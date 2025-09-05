---
masvs_category: MASVS-CRYPTO
platform: android
---

# APIs Criptográficas do Android

## Visão Geral

No capítulo ["Criptografia em Apps Móveis"](0x04g-Testing-Cryptography.md), introduzimos práticas recomendadas gerais de criptografia e descrevemos problemas típicos que podem ocorrer quando a criptografia é usada incorretamente. Neste capítulo, entraremos em mais detalhes sobre as APIs de criptografia do Android. Mostraremos como identificar o uso dessas APIs no código-fonte e como interpretar configurações criptográficas. Ao revisar código, compare sempre os parâmetros criptográficos utilizados com as práticas recomendadas atuais, conforme indicado neste guia.

Podemos identificar componentes-chave de um sistema criptográfico no Android:

- @MASTG-KNOW-0011
- @MASTG-KNOW-0043
- @MASTG-KNOW-0048

As APIs de criptografia do Android são baseadas na Java Cryptography Architecture (JCA). A JCA separa as interfaces e as implementações, tornando possível incluir vários [security providers](https://developer.android.com/reference/java/security/Provider.html "Android Security Providers") que podem implementar conjuntos de algoritmos criptográficos. A maioria das interfaces e classes da JCA está definida nos pacotes `java.security.*` e `javax.crypto.*`. Além disso, existem pacotes específicos do Android `android.security.*` e `android.security.keystore.*`.

KeyStore e KeyChain fornecem APIs para armazenar e usar chaves (nos bastidores, a API KeyChain usa o sistema KeyStore). Esses sistemas permitem administrar todo o ciclo de vida das chaves criptográficas. Requisitos e orientações para implementação de gerenciamento de chaves podem ser encontrados no [Key Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Key_Management_Cheat_Sheet.html "Key Management Cheat Sheet"). Podemos identificar as seguintes fases:

- gerar uma chave
- usar uma chave
- armazenar uma chave
- arquivar uma chave
- excluir uma chave

> Observe que o armazenamento de uma chave é analisado no capítulo ["Testando Armazenamento de Dados"](0x05d-Testing-Data-Storage.md).

Essas fases são gerenciadas pelo sistema Keystore/KeyChain. No entanto, como o sistema funciona depende de como o desenvolvedor do aplicativo o implementou. Para o processo de análise, concentre-se nas funções que são usadas pelo desenvolvedor. Você deve identificar e verificar as seguintes funções:

- @MASTG-KNOW-0012
- @MASTG-KNOW-0013
- Rotação de chaves

Apps que têm como alvo níveis modernos de API passaram pelas seguintes mudanças:

- Para Android 7.0 (API level 24) e superior [o Android Developer blog mostra que](https://android-developers.googleblog.com/2016/06/security-crypto-provider-deprecated-in.html "Security provider Crypto deprecated in Android N"):
    - Recomenda-se parar de especificar um security provider. Em vez disso, use sempre um @MASTG-KNOW-0011 atualizado.
    - O suporte ao provider `Crypto` foi removido e ele está obsoleto. O mesmo vale para seu `SHA1PRNG` para random seguro.
- Para Android 8.1 (API level 27) e superior a [Developer Documentation](https://developer.android.com/about/versions/oreo/android-8.1 "Cryptography updates") mostra que:
    - Conscrypt, conhecido como `AndroidOpenSSL`, é preferido em vez de usar Bouncy Castle e possui novas implementações: `AlgorithmParameters:GCM`, `KeyGenerator:AES`, `KeyGenerator:DESEDE`, `KeyGenerator:HMACMD5`, `KeyGenerator:HMACSHA1`, `KeyGenerator:HMACSHA224`, `KeyGenerator:HMACSHA256`, `KeyGenerator:HMACSHA384`, `KeyGenerator:HMACSHA512`, `SecretKeyFactory:DESEDE` e `Signature:NONEWITHECDSA`.
    - Você não deve mais usar `IvParameterSpec.class` para GCM, e sim `GCMParameterSpec.class`.
    - Sockets mudaram de `OpenSSLSocketImpl` para `ConscryptFileDescriptorSocket` e `ConscryptEngineSocket`.
    - `SSLSession` com parâmetros nulos gera `NullPointerException`.
    - É necessário ter arrays suficientemente grandes como bytes de entrada para gerar uma chave; caso contrário, uma `InvalidKeySpecException` é lançada.
    - Se a leitura de um Socket for interrompida, ocorre `SocketException`.
- Para Android 9 (API level 28) e superior o [Android Developer Blog](https://android-developers.googleblog.com/2018/03/cryptography-changes-in-android-p.html "Cryptography Changes in Android P") mostra ainda mais mudanças:
    - Você recebe um aviso se ainda especificar um security provider usando o método `getInstance` e tiver como alvo qualquer API abaixo de 28. Se o alvo for Android 9 (API level 28) ou superior, você recebe um erro.
    - O provider de segurança `Crypto` foi removido. Chamá-lo resultará em `NoSuchProviderException`.
- Para Android 10 (API level 29) a [Developer Documentation](https://developer.android.com/about/versions/10/behavior-changes-all#security "Security Changes in Android 10") lista todas as mudanças de segurança de rede.

**Recomendações Gerais:**

A lista a seguir deve ser considerada durante a análise do app:

- Garanta que as práticas recomendadas do capítulo ["Criptografia para Apps Móveis"](0x04g-Testing-Cryptography.md) sejam seguidas.
- Certifique-se de que o security provider esteja atualizado – [Updating security provider](https://developer.android.com/training/articles/security-gms-provider "Updating security provider").
- Pare de especificar um security provider e use a implementação padrão (AndroidOpenSSL, Conscrypt).
- Pare de usar o provider Crypto e seu `SHA1PRNG`, pois estão obsoletos.
- Especifique um security provider apenas para o sistema Android Keystore.
- Pare de usar cifras de criptografia baseadas em senha sem IV.
- Use KeyGenParameterSpec em vez de KeyPairGeneratorSpec.
