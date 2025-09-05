---
masvs_category: MASVS-AUTH
platform: ios
---

# Autenticação Local iOS

## Visão Geral

Durante a autenticação local, um aplicativo autentica o usuário contra credenciais armazenadas localmente no dispositivo. Em outras palavras, o usuário "desbloqueia" o aplicativo ou alguma camada interna de funcionalidade fornecendo um PIN, senha ou características biométricas válidas, como rosto ou impressão digital, que são verificadas referenciando dados locais. Geralmente, isso é feito para que os usuários possam retomar mais convenientemente uma sessão existente com um serviço remoto ou como meio de autenticação de step-up para proteger alguma função crítica.

Como afirmado anteriormente no capítulo ["Arquiteturas de Autenticação de Aplicativos Móveis"](0x04e-Testing-Authentication-and-Session-Management.md): O testador deve estar ciente de que a autenticação local deve sempre ser aplicada em um endpoint remoto ou baseada em uma primitiva criptográfica. Os atacantes podem facilmente contornar a autenticação local se nenhum dado retornar do processo de autenticação.

Uma variedade de métodos está disponível para integrar a autenticação local em aplicativos. O [framework Local Authentication](https://developer.apple.com/documentation/localauthentication "Local Authentication framework") fornece um conjunto de APIs para os desenvolvedores estenderem um diálogo de autenticação para um usuário. No contexto de conexão com um serviço remoto, é possível (e recomendado) aproveitar o [keychain](https://developer.apple.com/library/content/documentation/Security/Conceptual/keychainServConcepts/01introduction/introduction.html "Keychain Services") para implementar autenticação local.

A autenticação por impressão digital no iOS é conhecida como _Touch ID_. O sensor de ID de impressão digital é operado pelo [coprocessor de segurança SecureEnclave](https://www.blackhat.com/docs/us-16/materials/us-16-Mandt-Demystifying-The-Secure-Enclave-Processor.pdf "Demystifying the Secure Enclave Processor by Tarjei Mandt, Mathew Solnik, and David Wang") e não expõe dados de impressão digital para nenhuma outra parte do sistema. Além do Touch ID, a Apple introduziu o _Face ID_: que permite autenticação baseada em reconhecimento facial. Ambos usam APIs semelhantes em nível de aplicativo, o método real de armazenar os dados e recuperar os dados (por exemplo, dados faciais ou dados relacionados a impressões digitais é diferente).

Os desenvolvedores têm duas opções para incorporar autenticação Touch ID/Face ID:

- `LocalAuthentication.framework` é uma API de alto nível que pode ser usada para autenticar o usuário via Touch ID. O aplicativo não pode acessar nenhum dado associado à impressão digital cadastrada e é notificado apenas se a autenticação foi bem-sucedida.
- `Security.framework` é uma API de nível mais baixo para acessar [serviços de keychain](https://developer.apple.com/documentation/security/keychain_services "keychain Services"). Esta é uma opção segura se seu aplicativo precisar proteger alguns dados secretos com autenticação biométrica, já que o controle de acesso é gerenciado em nível de sistema e não pode ser facilmente contornado. `Security.framework` tem uma API C, mas existem vários [wrappers de código aberto disponíveis](https://www.raywenderlich.com/147308/secure-ios-user-data-keychain-touch-id "How To Secure iOS User Data: The keychain and Touch ID"), tornando o acesso ao keychain tão simples quanto ao NSUserDefaults. `Security.framework` subjaz ao `LocalAuthentication.framework`; a Apple recomenda usar APIs de nível mais alto sempre que possível.

Por favor, esteja ciente de que usar o `LocalAuthentication.framework` ou o `Security.framework` será um controle que pode ser contornado por um atacante, pois retorna apenas um booleano e nenhum dado para prosseguir. Veja [Don't touch me that way, by David Lindner et al](https://www.youtube.com/watch?v=XhXIHVGCFFM "Don\'t Touch Me That Way - David Lindner") para mais detalhes.