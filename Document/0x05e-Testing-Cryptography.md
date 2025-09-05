---
masvs_category: MASVS-CRYPTO
platform: android
---

# APIs Criptográficas Android

## Visão Geral

No capítulo ["Criptografia para Aplicativos Móveis"](0x04g-Testing-Cryptography.md), introduzimos as melhores práticas gerais de criptografia e descrevemos problemas típicos que podem ocorrer quando a criptografia é usada incorretamente. Neste capítulo, entraremos em mais detalhes sobre as APIs de criptografia do Android. Mostraremos como identificar o uso dessas APIs no código fonte e como interpretar configurações criptográficas. Ao revisar código, certifique-se de comparar os parâmetros criptográficos usados com as melhores práticas atuais, conforme vinculado neste guia.

Podemos identificar componentes-chave do sistema de criptografia no Android:

- @MASTG-KNOW-0011
- @MASTG-KNOW-0043
- @MASTG-KNOW-0048

As APIs de criptografia do Android são baseadas na Java Cryptography Architecture (JCA). A JCA separa as interfaces e a implementação, tornando possível incluir vários [provedores de segurança](https://developer.android.com/reference/java/security/Provider.html "Provedores de Segurança Android") que podem implementar conjuntos de algoritmos criptográficos. A maioria das interfaces e classes JCA são definidas nos pacotes `java.security.*` e `javax.crypto.*`. Além disso, existem pacotes Android específicos `android.security.*` e `android.security.keystore.*`.

KeyStore e KeyChain fornecem APIs para armazenar e usar chaves (nos bastidores, a API KeyChain usa o sistema KeyStore). Esses sistemas permitem administrar o ciclo de vida completo das chaves criptográficas. Requisitos e orientações para implementação de gerenciamento de chaves criptográficas podem ser encontrados na [Key Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Key_Management_Cheat_Sheet.html "Key Management Cheat Sheet"). Podemos identificar as seguintes fases:

- gerando uma chave
- usando uma chave
- armazenando uma chave
- arquivando uma chave
- excluindo uma chave

> Observe que o armazenamento de uma chave é analisado no capítulo ["Testando Armazenamento de Dados"](0x05d-Testing-Data-Storage.md).

Essas fases são gerenciadas pelo sistema Keystore/KeyChain. No entanto, como o sistema funciona depende de como o desenvolvedor do aplicativo o implementou. Para o processo de análise, você deve se concentrar nas funções que são usadas pelo desenvolvedor do aplicativo. Você deve identificar e verificar as seguintes funções:

- @MASTG-KNOW-0012
- @MASTG-KNOW-0013
- Rotação de chave

Aplicativos que direcionam níveis de API modernos, passaram pelas seguintes mudanças:

- Para Android 7.0 (API level 24) e superior [o blog do Android Developer mostra que](https://android-developers.googleblog.com/2016/06/security-crypto-provider-deprecated-in.html "Provedor de segurança Crypto obsoleto no Android N"):
    - É recomendado parar de especificar um provedor de segurança. Em vez disso, sempre use um @MASTG-KNOW-0011 corrigido.
    - O suporte para o provedor `Crypto` foi removido e o provedor está obsoleto. O mesmo se aplica ao seu `SHA1PRNG` para random seguro.
- Para Android 8.1 (API level 27) e superior a [Documentação do Desenvolvedor](https://developer.android.com/about/versions/oreo/android-8.1 "Atualizações de criptografia") mostra que:
    - Conscrypt, conhecido como `AndroidOpenSSL`, é preferido acima do uso de Bouncy Castle e tem novas implementações: `AlgorithmParameters:GCM`, `KeyGenerator:AES`, `KeyGenerator:DESEDE`, `KeyGenerator:HMACMD5`, `KeyGenerator:HMACSHA1`, `KeyGenerator:HMACSHA224`, `KeyGenerator:HMACSHA256`, `KeyGenerator:HMACSHA384`, `KeyGenerator:HMACSHA512`, `SecretKeyFactory:DESEDE` e `Signature:NONEWITHECDSA`.
    - Você não deve mais usar `IvParameterSpec.class` para GCM, mas usar `GCMParameterSpec.class` em vez disso.
    - Sockets mudaram de `OpenSSLSocketImpl` para `ConscryptFileDescriptorSocket` e `ConscryptEngineSocket`.
    - `SSLSession` com parâmetros nulos dá um `NullPointerException`.
    - Você precisa ter arrays grandes o suficiente como bytes de entrada para gerar uma chave, caso contrário, um `InvalidKeySpecException` é lançado.
    - Se uma leitura de Socket for interrompida, você obtém um `SocketException`.
- Para Android 9 (API level 28) e superior o [Android Developer Blog](https://android-developers.googleblog.com/2018/03/cryptography-changes-in-android-p.html "Mudanças de Criptografia no Android P") mostra ainda mais mudanças:
    - Você recebe um aviso se ainda especificar um provedor de segurança usando o método `getInstance` e você direciona qualquer API abaixo de 28. Se você direcionar Android 9 (API level 28) ou superior, você recebe um erro.
    - O provedor de segurança `Crypto` agora foi removido. Chamá-lo resultará em um `NoSuchProviderException`.
- Para Android 10 (API level 29) a [Documentação do Desenvolvedor](https://developer.android.com/about/versions/10/behavior-changes-all#security "Mudanças de Segurança no Android 10") lista todas as mudanças de segurança de rede.

**Recomendações Gerais:**

A seguinte lista de recomendações deve ser considerada durante o exame do aplicativo:

- Você deve garantir que as melhores práticas delineadas no capítulo ["Criptografia para Aplicativos Móveis"](0x04g-Testing-Cryptography.md) sejam seguidas.
- Você deve garantir que o provedor de segurança tenha as atualizações mais recentes - [Atualizando provedor de segurança](https://developer.android.com/training/articles/security-gms-provider "Atualizando provedor de segurança").
- Você deve parar de especificar um provedor de segurança e usar a implementação padrão (AndroidOpenSSL, Conscrypt).
- Você deve parar de usar o provedor de segurança Crypto e seu `SHA1PRNG`, pois estão obsoletos.
- Você deve especificar um provedor de segurança apenas para o sistema Android Keystore.
- Você deve parar de usar cifras de criptografia baseadas em senha sem IV.
- Você deve usar KeyGenParameterSpec em vez de KeyPairGeneratorSpec.