
---
masvs_category: MASVS-CRYPTO
platform: all
---

# Criptografia em Aplicativos Móveis

A criptografia desempenha um papel especialmente importante na segurança dos dados do usuário - ainda mais em um ambiente móvel, onde o acesso físico ao dispositivo do usuário por parte de atacantes é um cenário provável. Este capítulo fornece um esboço dos conceitos criptográficos e melhores práticas relevantes para aplicativos móveis. Essas melhores práticas são válidas independentemente do sistema operacional móvel.

## Conceitos Fundamentais

O objetivo da criptografia é fornecer confidencialidade, integridade de dados e autenticidade constantes, mesmo diante de um ataque. A confidencialidade envolve garantir a privacidade dos dados por meio do uso de criptografia. A integridade de dados trata da consistência dos dados e da detecção de adulteração e modificação de dados por meio do uso de hashing. A autenticidade garante que os dados venham de uma fonte confiável.

Um algoritmo de criptografia converte dados em texto simples (plaintext) em texto cifrado (ciphertext), que oculta o conteúdo original. Os dados em texto simples podem ser restaurados a partir do texto cifrado por meio da descriptografia. Existem dois tipos de criptografia: **simétrica** (criptografia e descriptografia usam a mesma chave secreta) e **assimétrica** (criptografia e descriptografia usam um par de chaves pública e privada). As operações de criptografia simétrica não protegem a integridade dos dados, a menos que sejam usadas com um modo de cifra aprovado que suporte criptografia autenticada com um vetor de inicialização (IV) aleatório que atenda ao requisito de "unicidade" [NIST SP 800-38D - "Recommendation for Block Cipher Modes of Operation: Galois/Counter Mode (GCM) and GMAC", 2007](https://csrc.nist.gov/pubs/sp/800/38/d/final).

**Algoritmos de criptografia de chave simétrica** usam a mesma chave para criptografia e descriptografia. Este tipo de criptografia é rápido e adequado para processamento de dados em massa. Como todos que têm acesso à chave são capazes de descriptografar o conteúdo criptografado, este método requer gerenciamento cuidadoso de chaves e controle centralizado sobre a distribuição de chaves.

**Algoritmos de criptografia de chave pública** operam com duas chaves separadas: a chave pública e a chave privada. A chave pública pode ser distribuída livremente, enquanto a chave privada não deve ser compartilhada com ninguém. Uma mensagem criptografada com a chave pública só pode ser descriptografada com a chave privada e vice-versa. Como a criptografia assimétrica é várias vezes mais lenta que as operações simétricas, normalmente é usada apenas para criptografar pequenas quantidades de dados, como chaves simétricas para criptografia em massa.

**Hashing** não é uma forma de criptografia, mas usa criptografia. As funções de hash mapeiam partes arbitrárias de dados em valores de comprimento fixo de forma determinística. Embora seja fácil calcular o hash a partir da entrada, é muito difícil (ou seja, inviável) determinar a entrada original a partir do hash. Além disso, o hash muda completamente quando até um único bit da entrada é alterado. As funções de hash são usadas para armazenar senhas, verificar integridade (por exemplo, assinaturas digitais ou gerenciamento de documentos) e gerenciar arquivos. Embora as funções de hash não forneçam uma garantia de autenticidade, elas podem ser combinadas como primitivas criptográficas para fazê-lo.

**Códigos de Autenticação de Mensagem (MACs)** combinam outros mecanismos criptográficos (como criptografia simétrica ou hashes) com chaves secretas para fornecer proteção de integridade e autenticidade. No entanto, para verificar um MAC, várias entidades devem compartilhar a mesma chave secreta e qualquer uma dessas entidades pode gerar um MAC válido. HMACs, o tipo mais comumente usado de MAC, dependem de hashing como primitiva criptográfica subjacente. O nome completo de um algoritmo HMAC geralmente inclui o tipo da função de hash subjacente (por exemplo, HMAC-SHA256 usa a função de hash SHA-256).

**Assinaturas** combinam criptografia assimétrica (ou seja, usando um par de chaves pública/privada) com hashing para fornecer integridade e autenticidade, criptografando o hash da mensagem com a chave privada. No entanto, ao contrário dos MACs, as assinaturas também fornecem a propriedade de não-repúdio, pois a chave privada deve permanecer exclusiva para o signatário dos dados.

**Funções de Derivação de Chave (KDFs)** derivam chaves secretas de um valor secreto (como uma senha) e são usadas para transformar chaves em outros formatos ou aumentar seu comprimento. Os KDFs são semelhantes às funções de hash, mas têm outros usos também (por exemplo, são usados como componentes de protocolos de acordo de chave multipartidária). Embora ambas as funções de hash e KDFs devam ser difíceis de reverter, os KDFs têm o requisito adicional de que as chaves que produzem devem ter um nível de aleatoriedade.

## Identificando Algoritmos Criptográficos Inseguros e/ou Obsoletos

Ao avaliar um aplicativo móvel, você deve garantir que ele não use algoritmos e protocolos criptográficos que tenham vulnerabilidades conhecidas significativas ou que sejam insuficientes para os requisitos modernos de segurança. Algoritmos que eram considerados seguros no passado podem se tornar inseguros com o tempo; portanto, é importante verificar periodicamente as melhores práticas atuais e ajustar as configurações de acordo.

Verifique se os algoritmos criptográficos estão atualizados e em conformidade com os padrões do setor. Algoritmos vulneráveis incluem cifras de bloco desatualizadas (como DES e 3DES), cifras de fluxo (como RC4), funções de hash (como MD5 e SHA1) e geradores de números aleatórios quebrados (como Dual_EC_DRBG e SHA1PRNG). Observe que mesmo algoritmos que são certificados (por exemplo, pelo NIST) podem se tornar inseguros com o tempo. Uma certificação não substitui a verificação periódica da solidez de um algoritmo. Algoritmos com vulnerabilidades conhecidas devem ser substituídos por alternativas mais seguras. Além disso, os algoritmos usados para criptografia devem ser padronizados e abertos à verificação. Criptografar dados usando algoritmos desconhecidos ou proprietários pode expor o aplicativo a diferentes ataques criptográficos que podem resultar na recuperação do texto simples.

Inspecione o código-fonte do aplicativo para identificar instâncias de algoritmos criptográficos conhecidos por serem fracos, como:

- [DES, 3DES](https://www.enisa.europa.eu/publications/algorithms-key-size-and-parameters-report-2014 "ENISA Algorithms, key size and parameters report 2014")
- RC2
- RC4
- [BLOWFISH](https://www.enisa.europa.eu/publications/algorithms-key-size-and-parameters-report-2014 "ENISA Algorithms, key size and parameters report 2014")
- MD4
- MD5
- SHA1

Os nomes das APIs criptográficas dependem da plataforma móvel específica.

Certifique-se de que:

- Os algoritmos criptográficos estão atualizados e em conformidade com os padrões do setor. Isso inclui, mas não se limita a cifras de bloco desatualizadas (por exemplo, DES), cifras de fluxo (por exemplo, RC4), bem como funções de hash (por exemplo, MD5) e geradores de números aleatórios quebrados como Dual_EC_DRBG (mesmo que sejam certificados pelo NIST). Todos esses devem ser marcados como inseguros e não devem ser usados e removidos do aplicativo e do servidor.
- Os comprimentos das chaves estão em conformidade com os padrões do setor e fornecem proteção suficiente por um longo período de tempo. Uma comparação de diferentes comprimentos de chave e a proteção que eles fornecem, levando em conta a Lei de Moore, está disponível [online](https://www.keylength.com/ "Keylength comparison").
- Através do [NIST SP 800-131A - "Transitioning the Use of Cryptographic Algorithms and Key Lengths", 2024](https://csrc.nist.gov/pubs/sp/800/131/a/r3/ipd), o NIST fornece recomendações e orientações sobre o alinhamento com recomendações futuras e a transição para chaves criptográficas mais fortes e algoritmos mais robustos.
- Os meios criptográficos não são misturados entre si: por exemplo, você não assina com uma chave pública ou tenta reutilizar um par de chaves usado para uma assinatura para fazer criptografia.
- Os parâmetros criptográficos são bem definidos dentro de uma faixa razoável. Isso inclui, mas não se limita a: salt criptográfico, que deve ter pelo menos o mesmo comprimento que a saída da função de hash, escolha razoável da função de derivação de senha e contagem de iterações (por exemplo, PBKDF2, scrypt ou bcrypt), IVs sendo aleatórios e únicos, modos de criptografia de bloco adequados ao propósito (por exemplo, ECB não deve ser usado, exceto em casos específicos), gerenciamento de chaves sendo feito corretamente (por exemplo, 3DES deve ter três chaves independentes) e assim por diante.

Algoritmos recomendados:

- Algoritmos de confidencialidade: AES-GCM-256 ou ChaCha20-Poly1305
- Algoritmos de integridade: SHA-256, SHA-384, SHA-512, BLAKE3, a família SHA-3
- Algoritmos de assinatura digital: RSA (3072 bits e superior), ECDSA com NIST P-384 ou EdDSA com Edwards448.
- Algoritmos de estabelecimento de chave: RSA (3072 bits e superior), DH (3072 bits ou superior), ECDH com NIST P-384

**Observação:** As recomendações são baseadas na percepção atual do setor do que é considerado apropriado. Elas estão alinhadas com as recomendações do NIST além de 2030, mas não levam necessariamente em conta os avanços na computação quântica. Para conselhos sobre criptografia pós-quântica, consulte a seção ["Pós-Quântica"](#post-quantum) abaixo.

Além disso, você deve sempre confiar em hardware seguro (se disponível) para armazenar chaves de criptografia, realizar operações criptográficas, etc.

Para mais informações sobre escolha de algoritmos e melhores práticas, consulte os seguintes recursos:

- ["Commercial National Security Algorithm Suite and Quantum Computing FAQ"](https://web.archive.org/web/20250305234320/https://cryptome.org/2016/01/CNSA-Suite-and-Quantum-Computing-FAQ.pdf "Commercial National Security Algorithm Suite and Quantum Computing FAQ")
- [NIST recommendations (2019)](https://www.keylength.com/en/4/ "NIST recommendations")
- [BSI recommendations (2019)](https://www.keylength.com/en/8/ "BSI recommendations")
- [NIST SP 800-56B Revision 2 - "Recommendation for Pair-Wise Key-Establishment Using Integer Factorization Cryptography", 2019](https://csrc.nist.gov/pubs/sp/800/56/b/r2/final): O NIST aconselha o uso de esquemas de transporte de chave baseados em RSA com um comprimento mínimo de módulo de pelo menos 2048 bits.
- [NIST SP 800-56A Revision 3 - "Recommendation for Pair-Wise Key-Establishment Schemes Using Discrete Logarithm Cryptography", 2018](https://csrc.nist.gov/pubs/sp/800/56/a/r3/final): O NIST aconselha o uso de esquemas de acordo de chave baseados em ECC, como Elliptic Curve Diffie-Hellman (ECDH), utilizando curvas de P-224 a P-521.
- [FIPS 186-5 - "Digital Signature Standard (DSS)", 2023](https://csrc.nist.gov/pubs/fips/186-5/final): O NIST aprova RSA, ECDSA e EdDSA para geração de assinatura digital. DSA deve ser usado apenas para verificar assinaturas previamente geradas.
- [NIST SP 800-186 - "Recommendations for Discrete Logarithm-Based Cryptography: Elliptic Curve Domain Parameters", 2023](https://csrc.nist.gov/pubs/sp/800/186/final): Fornece recomendações para parâmetros de domínio de curva elíptica usados em criptografia baseada em logaritmo discreto.

## Pós-Quântica

### Algoritmos de Criptografia de Chave Pública

Em 2024, o NIST aprovou o CRYSTALS-Kyber como um mecanismo de encapsulamento de chave (KEM) pós-quântico para estabelecer um segredo compartilhado em um canal público. Este segredo compartilhado pode então ser usado com algoritmos de chave simétrica para criptografia e descriptografia.

- [FIPS 203 - "Module-Lattice-Based Key-Encapsulation Mechanism Standard", 2024](https://csrc.nist.gov/pubs/fips/203/final): Especifica o CRYSTALS-Kyber como padrão para encapsulamento de chave pós-quântico.

### Assinaturas

Em 2024, o NIST aprovou o SLH-DSA e o ML-DSA como algoritmos de assinatura digital recomendados para geração e verificação de assinatura pós-quântica.

- [FIPS 205 - "Stateless Hash-Based Digital Signature Standard", 2024](https://csrc.nist.gov/pubs/fips/205/final): Especifica o SLH-DSA para assinaturas digitais pós-quânticas.
- [FIPS 204 - "Module-Lattice-Based Digital Signature Standard", 2024](https://csrc.nist.gov/pubs/fips/204/final): Especifica o ML-DSA para assinaturas digitais pós-quânticas.

## Problemas Comuns de Configuração

### Comprimento de Chave Insuficiente

Mesmo o algoritmo de criptografia mais seguro se torna vulnerável a ataques de força bruta quando esse algoritmo usa um tamanho de chave insuficiente.

Certifique-se de que o comprimento da chave atenda aos [padrões do setor aceitos](https://www.enisa.europa.eu/publications/algorithms-key-size-and-parameters-report-2014 "ENISA Algorithms, key size and parameters report 2014").

### Criptografia Simétrica com Chaves Criptográficas Embutidas no Código

A segurança da criptografia simétrica e dos hashes com chave (MACs) depende do sigilo da chave. Se a chave for divulgada, a segurança obtida pela criptografia é perdida. Para evitar isso, nunca armazene chaves secretas no mesmo local que os dados criptografados que elas ajudaram a criar. Um erro comum é criptografar dados armazenados localmente com uma chave de criptografia estática e embutida no código e compilar essa chave no aplicativo. Isso torna a chave acessível a qualquer pessoa que possa usar um desmontador.

Chave de criptografia embutida no código significa que uma chave é:

- parte dos recursos do aplicativo
- valor que pode ser derivado de valores conhecidos
- embutida no código

Primeiro, certifique-se de que nenhuma chave ou senha esteja armazenada dentro do código-fonte. Isso significa que você deve verificar código nativo, código JavaScript/Dart, código Java/Kotlin no Android e Objective-C/Swift no iOS. Observe que chaves embutidas no código são problemáticas mesmo se o código-fonte estiver ofuscado, pois a ofuscação é facilmente contornada por instrumentação dinâmica.

Se o aplicativo estiver usando TLS bidirecional (tanto certificados do servidor quanto do cliente são validados), certifique-se de que:

- A senha do certificado do cliente não esteja armazenada localmente ou esteja bloqueada no Keychain do dispositivo.
- O certificado do cliente não seja compartilhado entre todas as instalações.

Se o aplicativo depender de um contêiner criptografado adicional armazenado nos dados do aplicativo, verifique como a chave de criptografia é usada. Se um esquema de encapsulamento de chave for usado, certifique-se de que o segredo mestre seja inicializado para cada usuário ou que o contêiner seja recriptografado com uma nova chave. Se você puder usar o segredo mestre ou senha anterior para descriptografar o contêiner, verifique como as alterações de senha

são tratadas.

Chaves secretas devem ser armazenadas em armazenamento seguro do dispositivo sempre que a criptografia simétrica for usada em aplicativos móveis. Para obter mais informações sobre as APIs específicas da plataforma, consulte os capítulos "[Armazenamento de Dados no Android](0x05d-Testing-Data-Storage.md)" e "[Armazenamento de Dados no iOS](0x06d-Testing-Data-Storage.md)".

### Funções de Derivação de Chave Impropriamente Usadas

Algoritmos criptográficos (como criptografia simétrica ou alguns MACs) esperam uma entrada secreta de um determinado tamanho. Por exemplo, o AES usa uma chave de exatamente 16 bytes. Uma implementação nativa pode usar a senha fornecida pelo usuário diretamente como uma chave de entrada. Usar uma senha fornecida pelo usuário como uma chave de entrada tem os seguintes problemas:

- Se a senha for menor que a chave, o espaço completo da chave não é usado. O espaço restante é preenchido (espaços são às vezes usados para preenchimento).
- Uma senha fornecida pelo usuário realisticamente consistirá principalmente de caracteres exibíveis e pronunciáveis. Portanto, apenas alguns dos possíveis 256 caracteres ASCII são usados e a entropia é diminuída aproximadamente por um fator de quatro.

Certifique-se de que as senhas não sejam passadas diretamente para uma função de criptografia. Em vez disso, a senha fornecida pelo usuário deve ser passada para um KDF para criar uma chave criptográfica. Escolha uma contagem de iteração apropriada ao usar funções de derivação de senha. Por exemplo, [o NIST recomenda uma contagem de iteração de pelo menos 10.000 para PBKDF2](https://pages.nist.gov/800-63-3/sp800-63b.html#sec5 "NIST Special Publication 800-63B") e [para chaves críticas onde o desempenho percebido pelo usuário não é crítico, pelo menos 10.000.000](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-132.pdf "NIST Special Publication 800-132"). Para chaves críticas, é recomendado considerar a implementação de algoritmos reconhecidos pela [Password Hashing Competition (PHC)](https://password-hashing.net/ "PHC") como [Argon2](https://github.com/p-h-c/phc-winner-argon2 "Argon2").

### Geração Impropria de Números Aleatórios

Uma vulnerabilidade comum em aplicativos móveis é o uso inadequado de geradores de números aleatórios. Geradores de Números Pseudoaleatórios (PRNGs) regulares, embora suficientes para uso geral, não são projetados para fins criptográficos. Quando usados para gerar chaves, tokens ou outros valores críticos para segurança, eles podem tornar os sistemas vulneráveis à predição e ataque.

A questão fundamental é que dispositivos determinísticos não podem produzir verdadeira aleatoriedade. PRNGs simulam aleatoriedade usando algoritmos, mas sem entropia suficiente e força algorítmica, a saída pode ser previsível. Por exemplo, UUIDs podem parecer aleatórios, mas não fornecem entropia suficiente para uso seguro.

A abordagem correta é usar um [**Gerador de Números Pseudoaleatórios Criptograficamente Seguro (CSPRNG)**](https://en.wikipedia.org/wiki/Cryptographically_secure_pseudorandom_number_generator). CSPRNGs são projetados para resistir à análise estatística e predição, tornando-os adequados para gerar valores não adivinháveis. Todos os valores sensíveis à segurança devem ser gerados usando um CSPRNG com pelo menos 128 bits de entropia.

### Hashing Impropriamente Usado

Usar a função de hash errada para um determinado propósito pode comprometer tanto a segurança quanto a integridade dos dados. Cada função de hash é projetada com casos de uso específicos em mente, e aplicá-la incorretamente introduz risco.

Para verificações de integridade, escolha uma função de hash que ofereça forte resistência a colisões. Algoritmos como SHA-256, SHA-384, SHA-512, BLAKE3 e a família SHA-3 são apropriados para verificar integridade e autenticidade de dados. Evite algoritmos quebrados como MD5 ou SHA-1, pois são vulneráveis a ataques de colisão.

Não use funções de hash de propósito geral como SHA-2 ou SHA-3 para hashing de senhas ou derivação de chaves, especialmente com entrada previsível.

### Implementações Personalizadas de Criptografia

Inventar funções criptográficas proprietárias é demorado, difícil e provavelmente falhará. Em vez disso, podemos usar algoritmos bem conhecidos que são amplamente considerados seguros. Os sistemas operacionais móveis oferecem APIs criptográficas padrão que implementam esses algoritmos.

Inspecione cuidadosamente todos os métodos criptográficos usados dentro do código-fonte, especialmente aqueles que são aplicados diretamente a dados sensíveis. Todas as operações criptográficas devem usar APIs criptográficas padrão para Android e iOS (escreveremos sobre essas em mais detalhes nos capítulos específicos da plataforma). Qualquer operação criptográfica que não invoque rotinas padrão de provedores conhecidos deve ser inspecionada de perto. Preste muita atenção a algoritmos padrão que foram modificados. Lembre-se de que codificação não é o mesmo que criptografia! Sempre investigue mais quando encontrar operadores de manipulação de bits como XOR (OU exclusivo).

Em todas as implementações de criptografia, você precisa garantir que o seguinte sempre ocorra:

- Chaves de trabalho (como chaves intermediárias/derivadas em AES/DES/Rijndael) são adequadamente removidas da memória após o consumo ou em caso de erro.
- O estado interno de uma cifra deve ser removido da memória o mais rápido possível.

### Criptografia Impropriamente Implementada

O Advanced Encryption Standard (AES) é o padrão amplamente aceito para criptografia simétrica em aplicativos móveis. É uma cifra de bloco iterativa baseada em uma série de operações matemáticas vinculadas. O AES executa um número variável de rodadas na entrada, cada uma das quais envolve substituição e permutação dos bytes no bloco de entrada. Cada rodada usa uma chave de rodada de 128 bits que é derivada da chave AES original.

Até o momento desta escrita, nenhum ataque criptoanalítico eficiente contra o AES foi descoberto. No entanto, detalhes de implementação e parâmetros configuráveis, como o modo de cifra de bloco, deixam alguma margem para erro.

#### Modos de Cifra de Bloco Quebrados

A criptografia baseada em bloco é realizada em blocos de entrada discretos (por exemplo, o AES tem blocos de 128 bits). Se o texto simples for maior que o tamanho do bloco, o texto simples é dividido internamente em blocos do tamanho de entrada dado e a criptografia é realizada em cada bloco. Um modo de operação de cifra de bloco (ou modo de bloco) determina se o resultado da criptografia do bloco anterior impacta blocos subsequentes.

Evite usar o modo [ECB (Electronic Codebook)](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_Codebook_%28ECB%29 "Electronic Codebook (ECB)"). O ECB divide a entrada em blocos de tamanho fixo que são criptografados separadamente usando a mesma chave. Se vários blocos divididos contiverem o mesmo texto simples, eles serão criptografados em blocos de texto cifrado idênticos, o que torna os padrões nos dados mais fáceis de identificar. Em algumas situações, um atacante também pode ser capaz de reproduzir os dados criptografados.

<img src="Images/Chapters/0x07c/EncryptionMode.png" width="550px" />

Para novos projetos, prefira modos de criptografia autenticada com dados associados (AEAD), como Galois/Counter Mode (GCM) ou Counter with CBC-MAC (CCM), pois esses fornecem confidencialidade e integridade. Se GCM ou CCM não estiverem disponíveis, o modo Cipher Block Chaining (CBC) é melhor que o ECB, mas deve ser combinado com um HMAC e/ou garantir que nenhum erro seja dado, como "erro de preenchimento", "erro de MAC" ou "falha na descriptografia" para ser mais resistente a ataques de oráculo de preenchimento. No modo CBC, os blocos de texto simples são XORed com o bloco de texto cifrado anterior, garantindo que cada bloco criptografado seja único e randomizado, mesmo que os blocos contenham as mesmas informações.

Ao armazenar dados criptografados, recomendamos usar um modo de bloco que também proteja a integridade dos dados armazenados, como o Galois/Counter Mode (GCM). Este último tem o benefício adicional de que o algoritmo é obrigatório para cada implementação TLSv1.2 e, portanto, está disponível em todas as plataformas modernas. Para proteger a integridade e autenticidade dos dados usando o modo CBC, é recomendado combinar as técnicas do modo Counter (CTR) e do Cipher Block Chaining-Message Authentication Code (CBC-MAC) no que é chamado de modo CCM ([NIST, 2004](https://csrc.nist.gov/pubs/sp/800/38/c/upd1/final "NIST: Recommendation for Block Cipher Modes of Operation: the CCM Mode for Authentication and Confidentiality")).

Para mais informações sobre modos de bloco eficazes, consulte as [diretrizes do NIST sobre seleção de modo de bloco](https://csrc.nist.gov/groups/ST/toolkit/BCM/modes_development.html "NIST Modes Development, Proposed Modes").

#### Vetor de Inicialização Previsível

CBC, OFB, CFB, PCBC, GCM mode requerem um vetor de inicialização (IV) como entrada inicial para a cifra. O IV não precisa ser mantido em segredo, mas não deve ser previsível: deve ser aleatório e único/não repetível para cada mensagem criptografada. Certifique-se de que os IVs sejam gerados usando um gerador de números aleatórios criptograficamente seguro. Para mais informações sobre IVs, consulte o [artigo sobre vetores de inicialização do Crypto Fail](http://www.cryptofails.com/post/70059609995/crypto-noobs-1-initialization-vectors "Crypto Noobs #1: Initialization Vectors").

Preste atenção às bibliotecas criptográficas usadas no código: muitas bibliotecas de código aberto fornecem exemplos em suas documentações que podem seguir más práticas (por exemplo, usando um IV embutido no código). Um erro popular é copiar e colar código de exemplo sem alterar o valor do IV.

#### Usando a Mesma Chave para Criptografia e Autenticação

Um erro comum é reutilizar a mesma chave para criptografia CBC e CBC-MAC. A reutilização de chaves para diferentes propósitos geralmente não é recomendada, mas no caso do CBC-MAC o erro pode levar a um ataque MitM (["CBC-MAC", 2024.10.11](https://en.wikipedia.org/wiki/CBC-MAC "Wikipedia: CBC-MAC")).

#### Vetores de Inicialização em Modos de Operação com Estado

Observe que o uso de IVs é diferente ao usar os modos CTR e GCM, nos quais o vetor de inicialização é frequentemente um contador (no CTR combinado com um nonce). Então, aqui usar um IV previsível com seu próprio modelo stateful é exatamente o que é necessário. No CTR, você tem um novo nonce mais contador como entrada para cada nova operação de bloco. Por exemplo: para um texto simples de 5120 bits de comprimento: você tem 20 blocos, então precisa de 20 vetores de entrada consistindo de um nonce e contador. Já no GCM, você tem um único IV por operação criptográfica, que não deve ser repetido com a mesma chave. Consulte a seção 8 da [documentação do NIST sobre GCM](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf "Recommendation for Block Cipher Modes of Operation: Galois/Counter Mode and GMAC") para mais detalhes e recomendações do IV.

### Ataques de Oráculo de Preenchimento devido a Implementações de Preenchimento ou Operação de Bloco Mais Fracas

Nos velhos tempos, o preenchimento [PKCS1.5](https://tools.ietf.org/html/rfc2313 "PCKS1.5 in RFC2313") (em código: `PKCS1Padding`) era usado como um mecanismo de preenchimento ao fazer criptografia assimétrica. Este mecanismo é vulnerável ao ataque de oráculo de preenchimento. Portanto, é melhor usar OAEP (Optimal Asymmetric Encryption Padding) capturado em [PKCS#1 v2.0](https://tools.ietf.org/html/rfc2437 "PKCS1 v2.0 in RFC 2437") (em código: `OAEPPadding`, `OAEPwithSHA-256andMGF1Padding`, `OAEPwithSHA-224andMGF1Padding`, `OAEPwithSHA-384andMGF1Padding`, `OAEPwithSHA-512andMGF1Padding`). Observe que, mesmo ao usar OAEP, você ainda pode encontrar um problema conhecido como ataque de Manger, conforme descrito [no blog da Kudelskisecurity](https://research.kudelskisecurity.com/2018/04/05/breaking-rsa-oaep-with-mangers-attack/ "Kudelskisecurity").

Nota: AES-CBC com PKCS #7 mostrou ser vulnerável a ataques de oráculo de preenchimento também, desde que a implementação dê avisos, como "erro de preenchimento", "erro de MAC" ou "falha na descriptografia". Consulte [The Padding Oracle Attack](https://robertheaton.com/2013/07/29/padding-oracle-attack/ "The Padding Oracle Attack") e [The CBC Padding Oracle Problem](https://eklitzke.org/the-cbc-padding-oracle-problem "The CBC Padding Oracle Problem") para um exemplo. Em seguida, é melhor garantir que você adicione um HMAC após criptografar o texto simples: afinal, um texto cifrado com um MAC com falha não precisará ser descriptografado e pode ser descartado.

### Protegendo Chaves no Armazenamento e na Memória

Quando o dumping de memória faz parte do seu modelo de ameaça, as chaves podem ser acessadas no momento em que são usadas ativamente. O dumping de memória requer acesso root (por exemplo, um dispositivo root ou jailbroken) ou requer um aplicativo modificado com Frida (para que você possa usar ferramentas como @MASTG-TOOL-0106).
Portanto, é melhor considerar o seguinte, se as chaves ainda forem necessárias no dispositivo:

- **Chaves em um Servidor Remoto**: você pode usar cofres de chaves remotos, como Amazon KMS ou Azure Key Vault. Para alguns casos de uso, desenvolver uma camada de orquestração entre o aplicativo e o recurso remoto pode ser uma opção adequada. Por exemplo, uma função serverless em execução em um sistema Function as a Service (FaaS) (por exemplo, AWS Lambda ou Google Cloud Functions) que encaminha solicitações para recuperar uma chave de API ou segredo. Existem outras alternativas, como Amazon Cognito, Google Identity Platform ou Azure Active Directory.
- **Chaves dentro de Armazenamento com Backing de Hardware Seguro**: certifique-se de que todas as ações criptográficas e a própria chave permaneçam no Trusted Execution Environment (por exemplo, use [Android Keystore](https://developer.android.com/training/articles/keystore.html "Android keystore system")) ou [Secure Enclave](https://developer.apple.com/documentation/security/certificate_key_and_trust_services/keys/storing_keys_in_the_secure_enclave "Storing Keys in the Secure Enclave") (por exemplo, use o Keychain). Consulte os capítulos [Armazenamento de Dados no Android](0x05d-Testing-Data-Storage.md#storing-keys-using-hardware-backed-android-keystore) e [Armazenamento de Dados no iOS](0x06d-Testing-Data-Storage.md#the-keychain) para obter mais informações.
- **Chaves Protegidas por Criptografia de Envelope**: Se as chaves estiverem armazenadas fora do TEE / SE, considere usar criptografia multicamada: uma abordagem de _criptografia de envelope_ (consulte [OWASP Cryptographic Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html#encrypting-stored-keys "OWASP Cryptographic Storage Cheat Sheet: Encrypting Stored Keys"), [Google Cloud Key management guide](https://cloud.google.com/kms/docs/envelope-encryption?hl=en "Google Cloud Key management guide
: Envelope encryption"), [AWS Well-Architected Framework guide](https://docs.aws.amazon.com/wellarchitected/latest/financial-services-industry-lens/use-envelope-encryption-with-customer-master-keys.html "AWS Well-Architected Framework")), ou [uma abordagem HPKE](https://tools.ietf.org/html/draft-irtf-cfrg-hpke-08 "Hybrid Public Key Encryption") para criptografar chaves de criptografia de dados com chaves de criptografia de chaves.
- **Chaves na Memória**: certifique-se de que as chaves permaneçam na memória pelo menor tempo possível e considere zerar e anular chaves após operações criptográficas bem-sucedidas e em caso de erro. Nota: Em algumas linguagens e plataformas (como aquelas com coleta de lixo ou otimizações de gerenciamento de memória), zerar a memória de forma confiável pode não ser possível, pois o runtime pode mover ou copiar a memória ou atrasar a exclusão real. Para diretrizes gerais de cryptocoding, consulte [Clean memory of secret data](https://github.com/veorq/cryptocoding#clean-memory-of-secret-data/ "The Cryptocoding Guidelines by @veorq: Clean memory of secret data").

Nota: dada a facilidade de dumping de memória, nunca compartilhe a mesma chave entre contas e/ou dispositivos, exceto chaves públicas usadas para verificação de assinatura ou criptografia.

### Protegendo Chaves em Transporte

Quando as chaves precisam ser transportadas de um dispositivo para outro, ou do aplicativo para um backend, certifique-se de que a proteção adequada da chave esteja em vigor, por meio de um par de chaves de transporte ou outro mecanismo. Muitas vezes, as chaves são compartilhadas com métodos de ofuscação que podem ser facilmente revertidos. Em vez disso, certifique-se de que a criptografia assimétrica ou chaves de encapsulamento sejam usadas. Por exemplo, uma chave simétrica pode ser criptografada com a chave pública de um par de chaves assimétricas.

## APIs Criptográficas no Android e iOS

Embora os mesmos princípios criptográficos básicos se apliquem independentemente do sistema operacional específico, cada sistema operacional oferece sua própria implementação e APIs. As APIs criptográficas específicas da plataforma para armazenamento de dados são abordadas em maiores detalhes nos capítulos "[Armazenamento de Dados no Android](0x05d-Testing-Data-Storage.md)" e "[Testando Armazenamento de Dados no iOS](0x06d-Testing-Data-Storage.md)". A criptografia do tráfego de rede, especialmente Transport Layer Security (TLS), é abordada no capítulo "[APIs de Rede Android](0x05g-Testing-Network-Communication.md)".

## Política Criptográfica

Em organizações maiores, ou quando aplicativos de alto risco são criados, muitas vezes pode ser uma boa prática ter uma política criptográfica, baseada em frameworks como [NIST Recommendation for Key Management](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-57pt1r5.pdf "NIST 800-57 Rev5"). Quando erros básicos são encontrados na aplicação da criptografia, pode ser um bom ponto de partida para configurar uma política de lições aprendidas / gerenciamento de chaves criptográficas.

## Regulamentações de Criptografia

Quando você faz upload do aplicativo para a App Store ou Google Play, seu aplicativo é normalmente armazenado em um servidor dos EUA. Se o seu aplicativo contiver criptografia e for distribuído para qualquer outro país, é considerado uma exportação de criptografia. Isso significa que você precisa seguir as regulamentações de exportação de criptografia dos EUA. Além disso, alguns países têm regulamentações de importação para criptografia.

Saiba mais:

- [Complying with Encryption Export Regulations (Apple)](https://developer.apple.com/documentation/security/complying_with_encryption_export_regulations "Complying with Encryption Export Regulations")
- [Export compliance overview (Apple)](https://help.apple.com/app-store-connect/#/dev88f5c7bf9 "Export compliance overview")
- [Export compliance (Google)](https://support.google.com/googleplay/android-developer/answer/113770?hl=en "Export compliance")
- [Encryption and Export Administration Regulations (USA)](https://www.bis.doc.gov/index.php/policy-guidance/encryption "Encryption and Export Administration Regulations")
- [World map of encryption laws and policies](https://www.gp-digital.org/WORLD-MAP-OF-ENCRYPTION/)