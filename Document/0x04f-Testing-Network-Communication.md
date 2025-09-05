
---
masvs_category: MASVS-NETWORK
platform: all
---

# Comunicação de Rede de Aplicativos Móveis

Quase todos os aplicativos móveis conectados à rede dependem do Protocolo de Transferência de Hipertexto (HTTP) ou de sua versão segura, HTTPS (que usa Transport Layer Security, TLS) para trocar dados com endpoints remotos. Se não implementada com segurança, essa comunicação pode ser vulnerável a ataques baseados em rede, como sniffing de pacotes e ataques Machine-in-the-Middle (MITM). Neste capítulo, exploramos vulnerabilidades potenciais, técnicas de teste e melhores práticas para proteger a comunicação de rede de aplicativos móveis.

## Conexões Seguras

O tempo já passou desde que era razoável usar HTTP em texto claro sozinho e geralmente é trivial proteger conexões HTTP usando HTTPS. HTTPS é essencialmente HTTP em camadas sobre outro protocolo conhecido como Transport Layer Security (TLS). E o TLS executa um handshake usando criptografia de chave pública e, quando concluído, cria uma conexão segura.

Uma conexão HTTPS é considerada segura devido a três propriedades:

- **Confidencialidade:** O TLS criptografa os dados antes de enviá-los pela rede, o que significa que eles não podem ser lidos por um intermediário.
- **Integridade:** os dados não podem ser alterados sem detecção.
- **Autenticação:** o cliente pode validar a identidade do servidor para garantir que a conexão seja estabelecida com o servidor correto.

## Avaliação de Confiança do Servidor

Autoridades de Certificação (CAs) são uma parte integral de uma comunicação segura entre cliente e servidor e são predefinidas no armazenamento de confiança de cada sistema operacional. Por exemplo, no iOS existem mais de 200 certificados raiz instalados (veja [Documentação da Apple - Certificados raiz confiáveis disponíveis para sistemas operacionais Apple](https://support.apple.com/en-gb/HT204132 "Listas de certificados raiz confiáveis disponíveis no iOS")).

CAs podem ser adicionadas ao armazenamento de confiança, seja manualmente pelo usuário, por um MDM que gerencia o dispositivo corporativo ou através de malware. A questão então é: "você pode confiar em todas essas CAs e seu aplicativo deve confiar no armazenamento de confiança padrão?". Afinal, há casos bem conhecidos onde autoridades de certificação foram comprometidas ou enganadas para emitir certificados para impostores. Uma linha do tempo detalhada de violações e falhas de CA pode ser encontrada em [sslmate.com](https://sslmate.com/certspotter/failures "Timeline of PKI Security Failures").

Tanto o Android quanto o iOS permitem que o usuário instale CAs adicionais ou âncoras de confiança.

Um aplicativo pode querer confiar em um conjunto personalizado de CAs em vez do padrão da plataforma. As razões mais comuns para isso são:

- Conectar-se a um host com uma autoridade de certificação personalizada (uma CA que ainda não é conhecida ou confiável pelo sistema), como uma CA autoassinada ou emitida internamente dentro de uma empresa.
- Limitar o conjunto de CAs para uma lista específica de CAs confiáveis.
- Confiar em CAs adicionais não incluídas no sistema.

### Sobre Armazenamentos de Confiança

### Estendendo a Confiança

Sempre que o aplicativo se conectar a um servidor cujo certificado é autoassinado ou desconhecido pelo sistema, a conexão segura falhará. Este é tipicamente o caso de quaisquer CAs não públicas, por exemplo, aquelas emitidas por uma organização como um governo, corporação ou instituição de ensino para seu próprio uso.

Tanto o Android quanto o iOS oferecem meios para estender a confiança, ou seja, incluir CAs adicionais para que o aplicativo confie nas incorporadas do sistema mais as personalizadas.

No entanto, lembre-se de que os usuários do dispositivo sempre são capazes de incluir CAs adicionais. Portanto, dependendo do modelo de ameaça do aplicativo, pode ser necessário evitar confiar em quaisquer certificados adicionados ao armazenamento de confiança do usuário ou até ir além e confiar apenas em um certificado específico pré-definido ou conjunto de certificados.

Para muitos aplicativos, o "comportamento padrão" fornecido pela plataforma móvel será seguro o suficiente para seu caso de uso (no raro caso de uma CA confiável pelo sistema ser comprometida, os dados manipulados pelo aplicativo não são considerados sensíveis ou outras medidas de segurança são tomadas que são resilientes mesmo a tal violação de CA). No entanto, para outros aplicativos, como aplicativos financeiros ou de saúde, o risco de uma violação de CA, mesmo que raro, deve ser considerado.

### Restringindo a Confiança: Identity Pinning

Alguns aplicativos podem precisar aumentar ainda mais sua segurança restringindo o número de CAs em que confiam. Tipicamente, apenas as CAs que são usadas pelo desenvolvedor são explicitamente confiadas, enquanto todas as outras são desconsideradas. Esta restrição de confiança é conhecida como _Identity Pinning_, geralmente implementada como _Certificate Pinning_ ou _Public Key Pinning_.

> No OWASP MASTG nos referiremos a este termo como "Identity Pinning", "Certificate Pinning", "Public Key Pinning" ou simplesmente "Pinning".

Pinning é o processo de associar um endpoint remoto a uma identidade particular, como um certificado X.509 ou chave pública, em vez de aceitar qualquer certificado assinado por uma CA confiável. Após fixar a identidade do servidor (ou um determinado conjunto, também conhecido como _pinset_), o aplicativo móvel subsequentemente se conectará a esses endpoints remotos apenas se a identidade corresponder. Retirar a confiança de CAs desnecessárias reduz a superfície de ataque do aplicativo.

#### Diretrizes Gerais

A [OWASP Certificate Pinning Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Pinning_Cheat_Sheet.html) fornece orientação essencial sobre:

- quando o pinning é recomendado e quais exceções podem se aplicar.
- quando fixar: tempo de desenvolvimento (pré-carregamento) ou ao primeiro encontro (confiança no primeiro uso).
- o que fixar: certificado, chave pública ou hash.

Ambas as recomendações do Android e iOS correspondem ao "melhor caso" que é:

- Fixar apenas para endpoints remotos onde o desenvolvedor tem controle.
- no tempo de desenvolvimento via (NSC/ATS)
- fixar um hash do SPKI `subjectPublicKeyInfo`.

O pinning ganhou uma má reputação desde sua introdução há vários anos. Gostaríamos de esclarecer alguns pontos que são válidos pelo menos para segurança de aplicativos móveis:

- A má reputação é devido a razões operacionais (por exemplo, complexidade de implementação/gerenciamento de pin) não falta de segurança.
- Se um aplicativo não implementar pinning, isso não deve ser relatado como uma vulnerabilidade. No entanto, se o aplicativo deve verificar contra MAS-L2, deve ser implementado.
- Tanto o Android quanto o iOS tornam a implementação do pinning muito fácil e seguem as melhores práticas.
- O pinning protege contra uma CA comprometida ou uma CA maliciosa que está instalada no dispositivo. Nesses casos, o pinning impedirá que o SO estabeleça uma conexão segura com um servidor malicioso. No entanto, se um atacante estiver no controle do dispositivo, ele pode facilmente desativar qualquer lógica de pinning e, assim, ainda permitir que a conexão aconteça. Como resultado, isso não impedirá que um atacante acesse seu backend e abuse de vulnerabilidades do lado do servidor.
- Pinning em aplicativos móveis não é o mesmo que HTTP Public Key Pinning (HPKP). O cabeçalho HPKP não é mais recomendado em sites, pois pode levar os usuários a serem bloqueados do site sem qualquer maneira de reverter o bloqueio. Para aplicativos móveis, isso não é um problema, pois o aplicativo sempre pode ser atualizado através de um canal fora de banda (ou seja, a loja de aplicativos) caso haja algum problema.

#### Sobre Recomendações de Pinning em Desenvolvedores Android

O site [Android Developers](https://developer.android.com/training/articles/security-ssl#Pinning) inclui o seguinte aviso:

> Cuidado: Certificate Pinning não é recomendado para aplicativos Android devido ao alto risco de futuras mudanças de configuração do servidor, como mudar para outra Autoridade de Certificação, tornando o aplicativo incapaz de se conectar ao servidor sem receber uma atualização de software do cliente.

Eles também incluem esta [nota](https://developer.android.com/training/articles/security-config#CertificatePinning):

> Observe que, ao usar certificate pinning, você deve sempre incluir uma chave de backup para que, se for forçado a mudar para novas chaves ou alterar CAs (ao fixar em um certificado de CA ou um intermediário dessa CA), a conectividade do seu aplicativo não seja afetada. Caso contrário, você deve enviar uma atualização para o aplicativo para restaurar a conectividade.

A primeira declaração pode ser erroneamente interpretada como dizendo que eles "não recomendam certificate pinning". A segunda declaração esclarece isso: a recomendação real é que se os desenvolvedores quiserem implementar pinning, eles devem tomar as precauções necessárias.

#### Sobre Recomendações de Pinning em Desenvolvedores Apple

A Apple recomenda [pensar a longo prazo](https://developer.apple.com/news/?id=g9ejcf8y) e [criar uma estratégia adequada de autenticação de servidor](https://developer.apple.com/documentation/foundation/url_loading_system/handling_an_authentication_challenge/performing_manual_server_trust_authentication#2956135).

#### Recomendação OWASP MASTG

Pinning é uma prática recomendada, especialmente para aplicativos MAS-L2. No entanto, os desenvolvedores devem implementá-lo exclusivamente para os endpoints sob seu controle e ter certeza de incluir chaves de backup (também conhecidas como backup pins) e ter uma estratégia adequada de atualização do aplicativo.

#### Saiba mais

- ["Android Security: SSL Pinning"](https://appmattus.medium.com/android-security-ssl-pinning-1db8acb6621e)
- [OWASP Certificate Pinning Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Pinning_Cheat_Sheet.html)

## Verificando as Configurações TLS

Uma das funções principais do aplicativo móvel é enviar/receber dados por redes não confiáveis como a Internet. Se os dados não estiverem adequadamente protegidos em trânsito, um atacante com acesso a qualquer parte da infraestrutura de rede (por exemplo, um ponto de acesso Wi-Fi) pode interceptar, ler ou modificá-los. É por isso que protocolos de rede em texto claro raramente são aconselháveis.

A vasta maioria dos aplicativos depende do HTTP para comunicação com o backend. HTTPS envolve o HTTP em uma conexão criptografada (o acrônimo HTTPS originalmente se referia a HTTP sobre Secure Socket Layer (SSL); SSL é o predecessor obsoleto do TLS). O TLS permite a autenticação do serviço backend e garante confidencialidade e integridade dos dados de rede.

### Configurações TLS Recomendadas

Garantir a configuração TLS adequada no lado do servidor também é importante. O protocolo SSL está obsoleto e não deve mais ser usado.
Também TLS v1.0 e TLS v1.1 têm [vulnerabilidades conhecidas](https://portswigger.net/daily-swig/the-end-is-nigh-browser-makers-ditch-support-for-aging-tls-1-0-1-1-protocols "Browser-makers ditch support for aging TLS 1.0, 1.1 protocols") e seu uso está obsoleto em todos os principais navegadores até 2020.
TLS v1.2 e TLS v1.3 são considerados melhor prática para transmissão segura de dados. Começando com Android 10 (nível API 29) TLS v1.3 será ativado por padrão para comunicação mais rápida e segura. A [mudança principal com TLS v1.3](https://developer.android.com/about/versions/10/behavior-changes-all#tls-1.3 "TLS 1.3 enabled by default") é que personalizar suites de cifra não é mais possível e que todas elas são ativadas quando TLS v1.3 é ativado, enquanto o modo Zero Round Trip (0-RTT) não é suportado.

Quando tanto o cliente quanto o servidor são controlados pela mesma organização e usados apenas para comunicação entre si, você pode aumentar a segurança [reforçando a configuração](https://github.com/ssllabs/research/wiki/SSL-and-TLS-Deployment-Best-Practices "Qualys SSL/TLS Deployment Best Practices").

Se um aplicativo móvel se conecta a um servidor específico, sua pilha de rede pode ser ajustada para garantir o mais alto nível possível de segurança para a configuração do servidor. A falta de suporte no sistema operacional subjacente pode forçar o aplicativo móvel a usar uma configuração mais fraca.

### Terminologia de Suites de Cifra

Suites de cifra têm a seguinte estrutura:

```txt
Protocol_KeyExchangeAlgorithm_WITH_BlockCipher_IntegrityCheckAlgorithm
```

Esta estrutura inclui:

- Um **Protocolo** usado pela cifra
- Um **Algoritmo de Troca de Chaves** usado pelo servidor e pelo cliente para autenticar durante o handshake TLS
- Uma **Cifra de Bloco** usada para criptografar o fluxo de mensagens
- Um **Algoritmo de Verificação de Integridade** usado para autenticar mensagens

Exemplo: `TLS_RSA_WITH_3DES_EDE_CBC_SHA`

No exemplo acima, a suite de cifra usa:

- TLS como protocolo
- Criptografia assimétrica RSA para autenticação
- 3DES para criptografia simétrica com modo EDE_CBC
- Algoritmo de hash SHA para integridade

Note que no TLSv1.3 o Algoritmo de Troca de Chaves não faz parte da suite de cifra, em vez disso é determinado durante o handshake TLS.

Na listagem a seguir, apresentaremos os diferentes algoritmos de cada parte da suite de cifra.

**Protocolos:**

- `SSLv1`
- `SSLv2` - [RFC 6176](https://tools.ietf.org/html/rfc6176 "RFC 6176")
- `SSLv3` - [RFC 6101](https://tools.ietf.org/html/rfc6101 "RFC 6101")
- `TLSv1.0` - [RFC 2246](https://tools.ietf.org/rfc/rfc2246 "RFC 2246")
- `TLSv1.1` - [RFC 4346](https://tools.ietf.org/html/rfc4346 "RFC 4346")
- `TLSv1.2` - [RFC 5246](https://tools.ietf.org/html/rfc5246 "RFC 5246")
- `TLSv1.3` - [RFC 8446](https://tools.ietf.org/html/rfc8446 "RFC 8446")

**Algoritmos de Troca de Chaves:**

- `DSA` - [RFC 6979](https://tools.ietf.org/html/rfc6979 "RFC 6979")
- `ECDSA` - [RFC 6979](https://tools.ietf.org/html/rfc6979 "RFC 6979")
- `RSA` - [RFC 8017](https://tools.ietf.org/html/rfc8017 "RFC 8017")
- `DHE` - [RFC 2631](https://tools.ietf.org/html/rfc2631 "RFC 2631") - [RFC 7919](https://tools.ietf.org/html/rfc7919 "RFC 7919")
- `ECDHE` - [RFC 4492](https://tools.ietf.org/html/rfc4492 "RFC 4492")
- `PSK` - [RFC 4279](https://tools.ietf.org/html/rfc4279 "RFC 4279")
- `DSS` - [FIPS186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf "FIPS186-4")
- `DH_anon` - [RFC 2631](https://tools.ietf.org/html/rfc2631 "RFC 2631") - [RFC 7919](https://tools.ietf.org/html/rfc7919 "RFC 7919")
- `DHE_RSA` - [RFC 2631](https://tools.ietf.org/html/rfc2631 "RFC 2631") - [RFC 7919](https://tools.ietf.org/html/rfc7919 "RFC 7919")
- `DHE_DSS` - [RFC 2631](https://tools.ietf.org/html/rfc2631 "RFC 2631") - [RFC 7919](https://tools.ietf.org/html/rfc7919 "RFC 7919")
- `
`ECDHE_ECDSA` - [RFC 8422](https://tools.ietf.org/html/rfc8422 "RFC 8422")
- `ECDHE_PSK`  - [RFC 8422](https://tools.ietf.org/html/rfc8422 "RFC 8422") - [RFC 5489](https://tools.ietf.org/html/rfc5489 "RFC 5489")
- `ECDHE_RSA`  - [RFC 8422](https://tools.ietf.org/html/rfc8422 "RFC 8422")

**Cifras de Bloco:**

- `DES`  - [RFC 4772](https://tools.ietf.org/html/rfc4772 "RFC 4772")
- `DES_CBC`  - [RFC 1829](https://tools.ietf.org/html/rfc1829 "RFC 1829")
- `3DES`  - [RFC 2420](https://tools.ietf.org/html/rfc2420 "RFC 2420")
- `3DES_EDE_CBC` - [RFC 2420](https://tools.ietf.org/html/rfc2420 "RFC 2420")
- `AES_128_CBC` - [RFC 3268](https://tools.ietf.org/html/rfc3268 "RFC 3268")
- `AES_128_GCM`  - [RFC 5288](https://tools.ietf.org/html/rfc5288 "RFC 5288")
- `AES_256_CBC` - [RFC 3268](https://tools.ietf.org/html/rfc3268 "RFC 3268")
- `AES_256_GCM` - [RFC 5288](https://tools.ietf.org/html/rfc5288 "RFC 5288")
- `RC4_40`  - [RFC 7465](https://tools.ietf.org/html/rfc7465 "RFC 7465")
- `RC4_128`  - [RFC 7465](https://tools.ietf.org/html/rfc7465 "RFC 7465")
- `CHACHA20_POLY1305`  - [RFC 7905](https://tools.ietf.org/html/rfc7905 "RFC 7905") - [RFC 7539](https://tools.ietf.org/html/rfc7539 "RFC 7539")

**Algoritmos de Verificação de Integridade:**

- `MD5`  - [RFC 6151](https://tools.ietf.org/html/rfc6151 "RFC 6151")
- `SHA`  - [RFC 6234](https://tools.ietf.org/html/rfc6234 "RFC 6234")
- `SHA256`  - [RFC 6234](https://tools.ietf.org/html/rfc6234 "RFC 6234")
- `SHA384`  - [RFC 6234](https://tools.ietf.org/html/rfc6234 "RFC 6234")

Note que a eficiência de uma suite de cifra depende da eficiência de seus algoritmos.

Os seguintes recursos contêm as suites de cifra recomendadas mais recentes para usar com TLS:

- As suites de cifra recomendadas pela IANA podem ser encontradas em [TLS Cipher Suites](https://www.iana.org/assignments/tls-parameters/tls-parameters.xhtml#tls-parameters-4 "TLS Cipher Suites").
- As suites de cifra recomendadas pela OWASP podem ser encontradas na [TLS Cipher String Cheat Sheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/TLS_Cipher_String_Cheat_Sheet.md "OWASP TLS Cipher String Cheat Sheet").

Algumas versões do Android e iOS não suportam algumas das suites de cifra recomendadas, então para fins de compatibilidade você pode verificar as suites de cifra suportadas para [Android](https://developer.android.com/reference/javax/net/ssl/SSLSocket#cipher-suites "Cipher suites") e [iOS](https://developer.apple.com/documentation/security/1550981-ssl_cipher_suite_values?language=objc "SSL Cipher Suite Values") versões e escolher as suites de cifra suportadas principais.

Se você quiser verificar se seu servidor suporta as suites de cifra corretas, existem várias ferramentas que você pode usar:

- nscurl - veja [Comunicação de Rede iOS](0x06g-Testing-Network-Communication.md) para mais detalhes.
- [testssl.sh](https://github.com/drwetter/testssl.sh "testssl.sh") que "é uma ferramenta gratuita de linha de comando que verifica o serviço de um servidor em qualquer porta para o suporte de cifras TLS/SSL, protocolos, bem como algumas falhas criptográficas".

Finalmente, verifique se o servidor ou proxy de terminação no qual a conexão HTTPS termina está configurado de acordo com as melhores práticas. Veja também a [OWASP Transport Layer Protection cheat sheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.md "Transport Layer Protection Cheat Sheet") e as [Qualys SSL/TLS Deployment Best Practices](https://github.com/ssllabs/research/wiki/SSL-and-TLS-Deployment-Best-Practices "Qualys SSL/TLS Deployment Best Practices").

## Interceptando Tráfego de Rede Através de MITM

Interceptar o tráfego de aplicativos móveis é um aspecto crítico do teste de segurança, permitindo que testadores, analistas ou penetration testers analisem e manipulem comunicações de rede para identificar vulnerabilidades. Uma técnica chave neste processo é o ataque **Machine-in-the-Middle (MITM)** (também conhecido como ["Man-in-the-Middle"](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) (tradicionalmente), "Adversary-in-the-Middle" (por exemplo, por [MITRE](https://attack.mitre.org/techniques/T1638/) e [CAPEC](https://capec.mitre.org/data/definitions/94.html)), etc.), onde o _atacante_ posiciona sua máquina entre duas entidades em comunicação, tipicamente o aplicativo móvel (cliente) e os servidores com os quais está se comunicando. Ao fazer isso, a máquina do atacante intercepta e monitora os dados sendo transmitidos entre as diferentes partes.

Esta técnica é dupla:

- Tipicamente **usada por atacantes maliciosos** para interceptar, monitorar e potencialmente alterar a comunicação sem que qualquer uma das partes (aplicativo ou servidor) esteja ciente. Isso permite atividades maliciosas como eavesdropping, injetar conteúdo malicioso ou manipular os dados sendo trocados.
- No entanto, **no contexto do OWASP MASTG** e teste de segurança de aplicativos móveis, usamos como parte de nossas técnicas para permitir que o testador do aplicativo revise, analise ou modifique o tráfego para identificar vulnerabilidades como comunicação não criptografada ou controles de segurança fracos.

O método de interceptação específico usado depende dos mecanismos de segurança do aplicativo e da natureza dos dados sendo transmitidos. Cada abordagem varia em complexidade e eficácia, dependendo de fatores como criptografia e a capacidade do aplicativo de resistir à interferência.

Aqui está uma visão geral das técnicas de interceptação em diferentes camadas de rede:

| **Técnica de Interceptação** | **Ferramentas de Exemplo** | **Nota** |
|---------------------------|-------------------|-------------------|
| API hooking (`HttpUrlConnection`, `NSURLSession`, `WebRequest`) | Frida | Modifica como aplicativos lidam com solicitações de rede. |
| Hooking de funções TLS (`SSL_read`, `SSL_write`) | Frida, SSL Kill Switch | Intercepta dados criptografados antes que cheguem ao aplicativo. |
| Interceptação de proxy | Burp Suite, ZAP, mitmproxy | Requer que o aplicativo respeite configurações de proxy. |
| Sniffing de pacotes | `tcpdump`, Wireshark | Captura **todo** o tráfego TCP/UDP mas **não** descriptografa HTTPS. |
| MITM via ARP spoofing | bettercap | Engana dispositivos para enviar seu tráfego através da máquina do atacante mesmo quando a rede não é controlada pelo atacante. |
| AP Wi-Fi Rogue | `hostapd`, `dnsmasq`, `iptables`, `wpa_supplicant`, `airmon-ng` | Usa um ponto de acesso totalmente controlado pelo atacante. |

Você pode encontrar mais informações sobre estas técnicas em suas páginas de técnica correspondentes:

- @MASTG-TECH-0119
- @MASTG-TECH-0120
- @MASTG-TECH-0121
- @MASTG-TECH-0122
- @MASTG-TECH-0123
- @MASTG-TECH-0124

**Nota sobre certificate pinning:** Se o aplicativo usa certificate pinning, as técnicas acima podem parecer falhar uma vez que você começa a interceptar o tráfego, mas você pode contorná-lo usando métodos diferentes. Veja as seguintes técnicas para mais informações:

- Android: @MASTG-TECH-0012
- iOS: @MASTG-TECH-0064