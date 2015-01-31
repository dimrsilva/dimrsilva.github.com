---
layout: post
title: "Mudar SID do Windows Server 2012"
date: 2013-06-18 18:54
comments: true
categories: Windows, Server
---

Nas máquinas Windows, há uma chave especial chamada SID.
Quando utilizamos máquinas virtuais, existe a opção de clonar máquinas.
Essas máquinas clonadas recebem o mesmo SID da máquina original e acaba gerando problemas para entrarem no mesmo domínio.
Há um aplicativo que recria esse SID e pode ser usado para contornar esse problema.

```
C:\Windows\System32\Sysprep\sysprep.exe
```

Basta rodar o seguinte comando para que o SID seja recriado.
Deve ser escolhido a opção OOBE, e a opção de generalizar deve estar selecionada.



http://social.technet.microsoft.com/Forums/en-US/winserver8gen/thread/2eb10003-3f92-4549-b3d9-ef629bd9f3eb/

http://social.technet.microsoft.com/wiki/pt-br/contents/articles/14259.sysprep-no-windows-server-2012-e-windows-8.aspx
