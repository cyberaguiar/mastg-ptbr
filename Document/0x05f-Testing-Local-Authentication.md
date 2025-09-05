---
masvs_category: MASVS-AUTH
platform: android
---

# Autenticação Local Android

## Visão Geral

Durante a autenticação local, um aplicativo autentica o usuário contra credenciais armazenadas localmente no dispositivo. Em outras palavras, o usuário "desbloqueia" o aplicativo ou alguma camada interna de funcionalidade fornecendo um PIN, senha ou características biométricas válidas, como rosto ou impressão digital, que são verificadas referenciando dados locais. Geralmente, isso é feito para que os usuários possam retomar mais convenientemente uma sessão existente com um serviço remoto ou como meio de autenticação de step-up para proteger alguma função crítica.

Como afirmado anteriormente no capítulo ["Arquiteturas de Autenticação de Aplicativos Móveis"](0x04e-Testing-Authentication-and-Session-Management.md): O testador deve estar ciente de que a autenticação local deve sempre ser aplicada em um endpoint remoto ou baseada em um primitivo criptográfico. Os atacantes podem facilmente contornar a autenticação local se nenhum dado retornar do processo de autenticação.

No Android, existem dois mecanismos suportados pelo Android Runtime para autenticação local: o fluxo de Confirmação de Credenciais e o fluxo de Autenticação Biométrica.