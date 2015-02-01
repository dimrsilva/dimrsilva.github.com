---
layout: post
title: "Merge em arquivos de configuração do CodeIgniter"
date: 2013-07-24 22:48
comments: true
categories: [CodeIgniter, DRY, PHP]
keywords: CodeIgniter, Configuração, DRY, PHP
---

Ao carregar um arquivo de configuração no CodeIgniter, se existir tal arquivo
em uma sub-pasta com o nome do ambiente (development, production ou testing)
ele carrega esse arquivo, ao invés do primeiro.
Um exemplo é a seguinte estrutura de arquivos:

{% highlight bash %}
- config/
|  - production/
|  |  - config.php
|  - config.php
{% endhighlight %}

Em ambiente de produção, o CodeIgniter vai abrir o arquivo `config/production/config.php`,
nos demais ambientes, vai abrir o arquivo `config/config.php`. Isso é bom, mas pode ser melhor.

Imagine que este arquivo de configuração tenha muitas diretivas, todas as diretivas
deverão estar presente nos dois arquivos, pois o CodeIgniter abrirá apenas um deles.
Uma abordagem mais interessante seria se ele apenas fizesse o merge dos vetores de configuração,
bastando então definir apenas os itens que se diferenciam das configurações padrão.

Essa implementação é bem simples. Basta extender a classe de configuração (`CI_Config`) do CodeIgniter.

{% highlight php %}
<?php
class MY_Config extends CI_Config {

    /**
     * Load Config File
     *
     * @access  public
     * @param   string  the config file name
     * @param   boolean  if configuration values should be loaded into their own section
     * @param   boolean  true if errors should just return false, false if an error message should be displayed
     * @return  boolean if the file was loaded correctly
     */
    function load($file = '', $use_sections = FALSE, $fail_gracefully = FALSE)
    {
        $file = ($file == '') ? 'config' : str_replace('.php', '', $file);
        $found = FALSE;
        $loaded = FALSE;

        $check_locations = defined('ENVIRONMENT')
            ? array(ENVIRONMENT.'/'.$file, $file)
            : array($file);

        foreach ($this->_config_paths as $path)
        {
            foreach ($check_locations as $location)
            {
                $file_path = $path.'config/'.$location.'.php';

                if (in_array($file_path, $this->is_loaded, TRUE) OR !file_exists($file_path))
                {
                    continue;
                }

                $section = ($use_sections ? $file : false);
                if(!$this->load_file($file_path, $section, $fail_gracefully)) {
                    return false;
                }
            }
        }
        return TRUE;
    }

    private function load_file($file_path, $section, $fail_gracefully)
    {
        require $file_path;

        if ( ! isset($config) OR ! is_array($config))
        {
            if ($fail_gracefully === TRUE)
            {
                return FALSE;
            }
            show_error('Your '.$file_path.' file does not appear to contain a valid configuration array.');
        }

        if ($section)
        {
            if (isset($this->config[$section]))
            {
                $this->config[$section] = array_merge($this->config[$section], $config);
            }
            else
            {
                $this->config[$section] = $config;
            }
        }
        else
        {
            $this->config = array_merge($this->config, $config);
        }
        unset($config);
        $this->is_loaded[] = $file_path;
        log_message('debug', 'Config file loaded: '.$file_path);
        return true;
    }

}
{% endhighlight %}

Além disso, é preciso sobrescrever a função `get_config()`, definida no arquivo `Commom.php` do CodeIgniter. Para isso, basta definir ela antes que o CodeIgniter inicialize. Uma boa ideia é incluir um arquivo logo antes ao arquivo do CodeIgniter no arquivo `index.php` de sua aplicação.

{% highlight php %}
<?php
/* --------------------------------------------------------------------
 * LOAD APP BOOTSTRAP FILE
 * --------------------------------------------------------------------
 */
require_once APPPATH.'bootstrap.php';

/*
 * --------------------------------------------------------------------
 * LOAD THE BOOTSTRAP FILE
 * --------------------------------------------------------------------
 *
 * And away we go...
 *
 */
require_once BASEPATH.'core/CodeIgniter.php';
{% endhighlight %}

E adicionar no arquivo incluído (`APPPATH/bootstrap.php`), a nova definição da função.

{% highlight php %}
<?php

/**
* Loads the main config.php file
*
* This function lets us grab the config file even if the Config class
* hasn't been instantiated yet
*
* @access   private
* @return   array
*/
if ( ! function_exists('get_config'))
{
    function &get_config($replace = array())
    {
        static $_config;

        if (isset($_config))
        {
            return $_config[0];
        }

        // Is the config file in the environment folder?
        $file_path = APPPATH.'config/config.php';

        // Fetch the config file
        if ( ! file_exists($file_path))
        {
            exit('The configuration file does not exist.');
        }

        require($file_path);

        if ( defined('ENVIRONMENT') AND file_exists($environment_file_path = APPPATH.'config/'.ENVIRONMENT.'/config.php')) {
            require($environment_file_path);
        }

        // Does the $config array exist in the file?
        if ( ! isset($config) OR ! is_array($config))
        {
            exit('Your config file does not appear to be formatted correctly.');
        }

        // Are any values being dynamically replaced?
        if (count($replace) > 0)
        {
            foreach ($replace as $key => $val)
            {
                if (isset($config[$key]))
                {
                    $config[$key] = $val;
                }
            }
        }

        return $_config[0] =& $config;
    }
}
{% endhighlight %}
