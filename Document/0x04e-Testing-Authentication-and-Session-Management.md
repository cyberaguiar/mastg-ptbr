
---
masvs_category: MASVS-AUTH
platform: all
---

# Arquiteturas de Autenticação de Aplicativos Móveis

Problemas de autenticação e autorização são vulnerabilidades de segurança prevalentes. Na verdade, eles consistentemente ocupam o segundo lugar mais alto no [OWASP Top 10](https://owasp.org/www-project-top-ten/).

A maioria dos aplicativos móveis implementa algum tipo de autenticação de usuário. Embora parte da lógica de autenticação e gerenciamento de estado seja realizada pelo serviço de backend, a autenticação é uma parte tão integral da maioria das arquiteturas de aplicativos móveis que entender suas implementações comuns é importante.

Como os conceitos básicos são idênticos no iOS e Android, discutiremos arquiteturas de autenticação e autorização prevalentes e armadilhas neste guia genérico. Problemas específicos de autenticação do sistema operacional, como autenticação local e biométrica, serão discutidos nos respectivos capítulos específicos do sistema operacional.

## Pressupostos Gerais

### Autenticação Adequada Está em Vigor

Execute as seguintes etapas ao testar autenticação e autorização:

- Identifique os fatores de autenticação adicionais que o aplicativo usa.
- Localize todos os endpoints que fornecem funcionalidade crítica.
- Verifique se os fatores adicionais são estritamente aplicados em todos os endpoints do lado do servidor.

Vulnerabilidades de bypass de autenticação existem quando o estado de autenticação não é consistentemente aplicado no servidor e quando o cliente pode adulterar o estado. Enquanto o serviço de backend está processando solicitações do cliente móvel, ele deve aplicar consistentemente verificações de autorização: verificando que o usuário está logado e autorizado toda vez que um recurso é solicitado.

Considere o seguinte exemplo do [OWASP Web Testing Guide](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/04-Authentication_Testing/04-Testing_for_Bypassing_Authentication_Schema "Testing for Bypassing Authentication Schema (WSTG-ATHN-04)"). No exemplo, um recurso web é acessado através de uma URL, e o estado de autenticação é passado através de um parâmetro GET:

```html
http://www.site.com/page.asp?authenticated=no
```

O cliente pode alterar arbitrariamente os parâmetros GET enviados com a solicitação. Nada impede o cliente de simplesmente mudar o valor do parâmetro `authenticated` para "sim", efetivamente contornando a autenticação.

Embora este seja um exemplo simplista que você provavelmente não encontrará na prática, programadores às vezes confiam em parâmetros "ocultos" do lado do cliente, como cookies, para manter o estado de autenticação. Eles assumem que esses parâmetros não podem ser adulterados. Considere, por exemplo, a seguinte [vulnerabilidade clássica no Nortel Contact Center Manager](http://seclists.org/bugtraq/2009/May/251 "SEC Consult SA-20090525-0 :: Nortel Contact Center Manager Server Authentication Bypass Vulnerability"). A aplicação web administrativa do appliance da Nortel confiava no cookie "isAdmin" para determinar se o usuário logado deveria receber privilégios administrativos. Consequentemente, era possível obter acesso de administrador simplesmente definindo o valor do cookie como segue:

```html
isAdmin=True
```

Especialistas em segurança costumavam recomendar usar autenticação baseada em sessão e manter dados de sessão apenas no servidor. Isso impede qualquer forma de adulteração do lado do cliente com o estado da sessão. No entanto, o ponto principal de usar autenticação stateless em vez de autenticação baseada em sessão é _não_ ter estado de sessão no servidor. Em vez disso, o estado é armazenado em tokens do lado do cliente e transmitido com cada solicitação. Nesse caso, ver parâmetros do lado do cliente como `isAdmin` é perfeitamente normal.

Para prevenir adulteração, assinaturas criptográficas são adicionadas a tokens do lado do cliente. Claro, as coisas podem dar errado, e implementações populares de autenticação stateless foram vulneráveis a ataques. Por exemplo, a verificação de assinatura de algumas implementações de JSON Web Token (JWT) poderia ser desativada [definindo o tipo de assinatura para "None"](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/ "Critical vulnerabilities in JSON Web Token libraries").

### Melhores Práticas para Senhas

A força da senha é uma preocupação chave quando senhas são usadas para autenticação. A política de senha define requisitos aos quais os usuários finais devem aderir. Uma política de senha normalmente especifica comprimento da senha, complexidade da senha e topologias de senha. Uma política de senha "forte" torna a quebra de senha manual ou automatizada difícil ou impossível. Para mais informações, consulte a [OWASP Authentication Cheat Sheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Authentication_Cheat_Sheet.md#implement-proper-password-strength-controls "Implement Proper Password Strength Controls").

## Diretrizes Gerais sobre Teste de Autenticação

Não há uma abordagem única para autenticação. Ao revisar a arquitetura de autenticação de um aplicativo, você deve primeiro considerar se o(s) método(s) de autenticação usados são apropriados no contexto dado. A autenticação pode ser baseada em um ou mais dos seguintes:

- Algo que o usuário sabe (senha, PIN, padrão, etc.)
- Algo que o usuário tem (cartão SIM, gerador de senha única ou token de hardware)
- Uma propriedade biométrica do usuário (impressão digital, retina, voz)

O número de procedimentos de autenticação implementados por aplicativos móveis depende da sensibilidade das funções ou recursos acessados. Consulte as melhores práticas da indústria ao revisar funções de autenticação. Autenticação de nome de usuário/senha (combinada com uma política de senha razoável) é geralmente considerada suficiente para aplicativos que têm um login de usuário e não são muito sensíveis. Esta forma de autenticação é usada pela maioria dos aplicativos de mídia social.

Para aplicativos sensíveis, adicionar um segundo fator de autenticação é geralmente apropriado. Isso inclui aplicativos que fornecem acesso a informações muito sensíveis (como números de cartão de crédito) ou permitem que usuários transfiram fundos. Em algumas indústrias, esses aplicativos também devem cumprir certos padrões. Por exemplo, aplicativos financeiros devem garantir conformidade com o Payment Card Industry Data Security Standard (PCI DSS), o Gramm Leach Bliley Act e o Sarbanes-Oxley Act (SOX). Considerações de conformidade para o setor de saúde dos EUA incluem o Health Insurance Portability and Accountability Act (HIPAA) e o Patient Safety Rule.

## Autenticação Stateful vs. Stateless

Você geralmente encontrará que o aplicativo móvel usa HTTP como camada de transporte. O protocolo HTTP em si é stateless, então deve haver uma maneira de associar as solicitações HTTP subsequentes do usuário a esse usuário. Caso contrário, as credenciais de login do usuário teriam que ser enviadas com cada solicitação. Além disso, tanto o servidor quanto o cliente precisam acompanhar os dados do usuário (por exemplo, os privilégios ou função do usuário). Isso pode ser feito de duas maneiras diferentes:

- Com autenticação _stateful_, um ID de sessão único é gerado quando o usuário faz login. Em solicitações subsequentes, este ID de sessão serve como referência aos detalhes do usuário armazenados no servidor. O ID de sessão é _opaco_; não contém nenhum dado do usuário.

- Com autenticação _stateless_, todas as informações de identificação do usuário são armazenadas em um token do lado do cliente. O token pode ser passado para qualquer servidor ou micro serviço, eliminando a necessidade de manter estado de sessão no servidor. A autenticação stateless é frequentemente fatorada para um servidor de autorização, que produz, assina e opcionalmente criptografa o token após o login do usuário.

Aplicações web comumente usam autenticação stateful com um ID de sessão aleatório que é armazenado em um cookie do lado do cliente. Embora aplicativos móveis às vezes usem sessões stateful de forma similar, abordagens baseadas em token stateless estão se tornando populares por várias razões:

- Elas melhoram a escalabilidade e o desempenho eliminando a necessidade de armazenar estado de sessão no servidor.
- Tokens permitem que desenvolvedores desacoplem a autenticação do aplicativo. Tokens podem ser gerados por um servidor de autenticação, e o esquema de autenticação pode ser alterado de forma transparente.

Como testador de segurança móvel, você deve estar familiarizado com ambos os tipos de autenticação.

### Autenticação Stateful

Autenticação stateful (ou "baseada em sessão") é caracterizada por registros de autenticação tanto no cliente quanto no servidor. O fluxo de autenticação é o seguinte:

1. O aplicativo envia uma solicitação com as credenciais do usuário para o servidor backend.
2. O servidor verifica as credenciais. Se as credenciais forem válidas, o servidor cria uma nova sessão junto com um ID de sessão aleatório.
3. O servidor envia ao cliente uma resposta que inclui o ID de sessão.
4. O cliente envia o ID de sessão com todas as solicitações subsequentes. O servidor valida o ID de sessão e recupera o registro de sessão associado.
5. Após o logout do usuário, o registro de sessão do lado do servidor é destruído e o cliente descarta o ID de sessão.

Quando as sessões são gerenciadas inadequadamente, elas são vulneráveis a uma variedade de ataques que podem comprometer a sessão de um usuário legítimo, permitindo que o atacante se passe pelo usuário. Isso pode resultar em perda de dados, comprometimento de confidencialidade e ações ilegítimas.

**Melhores Práticas:**

Localize quaisquer endpoints do lado do servidor que forneçam informações ou funções sensíveis e verifique a aplicação consistente de autorização. O serviço backend deve verificar o ID de sessão ou token do usuário e garantir que o usuário tenha privilégios suficientes para acessar o recurso. Se o ID de sessão ou token estiver faltando ou for inválido, a solicitação deve ser rejeitada.

Certifique-se de que:

- IDs de sessão são gerados aleatoriamente no lado do servidor.
- Os IDs não podem ser facilmente adivinhados (use comprimento e entropia adequados).
- IDs de sessão são sempre trocados sobre conexões seguras (por exemplo, HTTPS).
- O aplicativo móvel não salva IDs de sessão em armazenamento permanente.
- O servidor verifica a sessão sempre que um usuário tenta acessar elementos privilegiados do aplicativo (um ID de sessão deve ser válido e deve corresponder ao nível de autorização adequado).
- A sessão é encerrada no lado do servidor e as informações da sessão são excluídas dentro do aplicativo móvel após expirar ou o usuário fazer logout.

A autenticação não deve ser implementada do zero, mas construída sobre estruturas comprovadas. Muitas estruturas populares fornecem funcionalidade de autenticação e gerenciamento de sessão prontas. Se o aplicativo usar APIs de estrutura para autenticação, verifique a documentação de segurança da estrutura para melhores práticas. Guias de segurança para estruturas comuns estão disponíveis nos seguintes links:

- [Spring (Java)](https://projects.spring.io/spring-security "Spring (Java)")
- [Struts (Java)](https://struts.apache.org/security/ "Struts (Java)")
- [Laravel (PHP)](https://laravel.com/docs/9.x/authentication "Laravel (PHP)")
- [Ruby on Rails](https://guides.rubyonrails.org/security.html "Ruby on Rails")
- [ASP.Net](https://learn.microsoft.com/en-us/aspnet/core/security "ASP.NET")

Um ótimo recurso para testar autenticação do lado do servidor é o OWASP Web Testing Guide, especificamente os capítulos [Testing Authentication](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/04-Authentication_Testing/README) e [Testing Session Management](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/06-Session_Management_Testing/README).

### Autenticação Stateless

Autenticação baseada em token é implementada enviando um token assinado (verificado pelo servidor) com cada solicitação HTTP. O formato de token mais comumente usado é o JSON Web Token, definido em [RFC7519](https://tools.ietf.org/html/rfc7519 "RFC7519"). Um JWT pode codificar o estado completo da sessão como um objeto JSON. Portanto, o servidor não precisa armazenar nenhum dado de sessão ou informação de autenticação.

Tokens JWT consistem em três partes codificadas em Base64Url separadas por pontos. A estrutura do Token é a seguinte:

```default
base64UrlEncode(header).base64UrlEncode(payload).base64UrlEncode(signature)
```

O seguinte exemplo mostra um [JSON Web Token codificado em Base64Url](https://jwt.io/#debugger "JWT Example on jwt.io"):

```default
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6Ikpva
G4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

O _header_ normalmente consiste em duas partes: o tipo de token, que é JWT, e o algoritmo de hash sendo usado para computar a assinatura. No exemplo acima, o header decodifica da seguinte forma:

```json
{"alg":"HS256","typ":"JWT"}
```

A segunda parte do token é o _payload_, que contém as chamadas claims. Claims são declarações sobre uma entidade (tipicamente, o usuário) e metadados adicionais. Por exemplo:

```json
{"sub":"1234567890","name":"John Doe","admin":true}
```

A assinatura é criada aplicando o algoritmo especificado no header JWT ao header codificado, payload codificado e um valor secreto. Por exemplo, ao usar o algoritmo HMAC SHA256 a assinatura é criada da seguinte maneira:

```java
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

Note que o segredo é compartilhado entre o servidor de autenticação e o serviço backend - o cliente não o conhece. Isso prova que o token foi obtido de um serviço de autenticação legítimo. Também impede que o cliente adultere as claims contidas no token.

**Melhores Práticas:**

Verifique se a implementação adere às [melhores práticas](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html) do JWT:

- Verifique se o HMAC é verificado para todas as solicitações de entrada contendo um token.
- Verifique se a chave de assinatura privada ou chave secreta HMAC nunca é compartilhada com o cliente. Ela deve estar disponível apenas para o emissor e verificador.
- Verifique se nenhum dado sensível, como informações pessoais identificáveis, está embutido no JWT. Por exemplo, decodificando o JWT codificado em base64 e descobrindo que tipo de dados ele transmite e se esses dados são criptografados. Se, por alguma razão, a arquitetura exigir a transmissão de tais informações no token, certifique-se de que a criptografia de payload está sendo aplicada.
- Certifique-se de que ataques de replay são tratados com a claim `jti` (JWT ID), que dá ao JWT um identificador único.
- Certifique-se de que ataques de relay entre serviços são tratados com a claim `aud` (audience), que define para qual aplicação o token é destinado.
- Verifique se os tokens são armazenados com segurança no telefone móvel, com, por exemplo, KeyChain (iOS) ou KeyStore (Android).
- Verifique se o algoritmo de hash é aplicado. Um ataque comum inclui alterar o token para usar uma assinatura vazia (por exemplo, signature = "") e definir o algoritmo de assinatura para `none`, indicando que "a integridade do token já foi verificada". Algumas bibliotecas podem tratar

tokens assinados com o algoritmo `none` como se fossem tokens válidos com assinaturas verificadas, então o aplicativo confiará em claims de token alteradas.
- Verifique se os tokens incluem uma [claim de expiração "exp"](https://tools.ietf.org/html/rfc7519#section-4.1.4 "RFC 7519") e o backend não processa tokens expirados. Um método comum de conceder tokens combina [tokens de acesso e tokens de atualização](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/ "Refresh tokens & access tokens"). Quando o usuário faz login, o serviço backend emite um _token de acesso_ de curta duração e um _token de atualização_ de longa duração. O aplicativo pode então usar o token de atualização para obter um novo token de acesso, se o token de acesso expirar.

Existem dois plugins Burp diferentes que podem ajudá-lo a testar as vulnerabilidades listadas acima:

- [JSON Web Token Attacker](https://portswigger.net/bappstore/82d6c60490b540369d6d5d01822bdf61 "JSON Web Token Attacker")
- [JSON Web Tokens](https://portswigger.net/bappstore/f923cbf91698420890354c1d8958fee6 "JSON Web Tokens")

Além disso, certifique-se de verificar a [OWASP JWT Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html "JSON Web Token (JWT) Cheat Sheet for Java") para informações adicionais.

## OAuth 2.0

[OAuth 2.0](https://oauth.net/articles/authentication/ "OAuth 2.0") é um framework de autorização que permite que aplicações de terceiros obtenham acesso limitado a contas de usuário em serviços HTTP remotos, como APIs e aplicações habilitadas para web.

Usos comuns para OAuth2 incluem:

- Obter permissão do usuário para acessar um serviço online usando sua conta.
- Autenticar em um serviço online em nome do usuário.
- Lidar com erros de autenticação.

De acordo com o OAuth 2.0, um cliente móvel que busca acesso aos recursos de um usuário deve primeiro pedir ao usuário para autenticar em um _servidor de autenticação_. Com a aprovação dos usuários, o servidor de autorização então emite um token que permite que o aplicativo aja em nome do usuário. Note que a especificação OAuth2 não define nenhum tipo particular de autenticação ou formato de token de acesso.

### Visão Geral do Protocolo

OAuth 2.0 define quatro papéis:

- Proprietário do Recurso: o proprietário da conta
- Cliente: a aplicação que quer acessar a conta do usuário com os tokens de acesso
- Servidor de Recurso: hospeda as contas de usuário
- Servidor de Autorização: verifica a identidade do usuário e emite tokens de acesso para a aplicação

Nota: A API cumpre ambos os papéis de Servidor de Recurso e Servidor de Autorização. Portanto, nos referiremos a ambos como a API.

<img src="Images/Chapters/0x04e/abstract_oath2_flow.png" width="400px" />

Aqui está uma [explicação mais detalhada](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2 "An Introduction into OAuth2") das etapas no diagrama:

1. A aplicação solicita autorização do usuário para acessar recursos do serviço.
2. Se o usuário autorizar a solicitação, a aplicação recebe uma concessão de autorização. A concessão de autorização pode tomar várias formas (explícita, implícita, etc.).
3. A aplicação solicita um token de acesso do servidor de autorização (API) apresentando autenticação de sua própria identidade junto com a concessão de autorização.
4. Se a identidade da aplicação for autenticada e a concessão de autorização for válida, o servidor de autorização (API) emite um token de acesso para a aplicação, completando o processo de autorização. O token de acesso pode ter um token de atualização companheiro.
5. A aplicação solicita o recurso do servidor de recurso (API) e apresenta o token de acesso para autenticação. O token de acesso pode ser usado de várias maneiras (por exemplo, como um token bearer).
6. Se o token de acesso for válido, o servidor de recurso (API) serve o recurso para a aplicação.

No OAuth2, o _agente do usuário_ é a entidade que realiza a autenticação. A autenticação OAuth2 pode ser realizada através de um agente de usuário externo (por exemplo, Chrome ou Safari) ou no próprio aplicativo (por exemplo, através de um WebView incorporado no aplicativo ou uma biblioteca de autenticação). Nenhum dos dois modos é intrinsecamente "melhor" que o outro. A escolha depende do caso de uso específico do aplicativo e do modelo de ameaça.

**Agente de Usuário Externo:** Usar um _agente de usuário externo_ é o método de escolha para aplicativos que precisam interagir com contas de mídia social (Facebook, Twitter, etc.). Vantagens deste método incluem:

- As credenciais do usuário nunca são expostas diretamente ao aplicativo. Isso garante que o aplicativo não possa obter as credenciais durante o processo de login ("phishing de credenciais").
- Quase nenhuma lógica de autenticação deve ser adicionada ao próprio aplicativo, prevenindo erros de codificação.

No lado negativo, não há como controlar o comportamento do navegador (por exemplo, para ativar o certificate pinning).

**Agente de Usuário Incorporado:** Usar um _agente de usuário incorporado_ é o método de escolha para aplicativos que precisam operar dentro de um ecossistema fechado, por exemplo para interagir com contas corporativas. Por exemplo, considere um aplicativo bancário que usa OAuth2 para recuperar um token de acesso do servidor de autenticação do banco, que é então usado para acessar vários micro serviços. Nesse caso, phishing de credenciais não é um cenário viável. É provavelmente preferível manter o processo de autenticação no (esperançosamente) aplicativo bancário cuidadosamente protegido, em vez de colocar confiança em componentes externos.

### Melhores Práticas

Para melhores práticas adicionais e informações detalhadas, consulte os seguintes documentos de origem:

- [RFC6749 - The OAuth 2.0 Authorization Framework (October 2012)](https://tools.ietf.org/html/rfc6749)
- [RFC8252 - OAuth 2.0 for Native Apps (October 2017)](https://tools.ietf.org/html/rfc8252)
- [RFC6819 - OAuth 2.0 Threat Model and Security Considerations (January 2013)](https://tools.ietf.org/html/rfc6819)

Algumas das melhores práticas incluem, mas não estão limitadas a:

- **Agente do usuário:**
    - O usuário deve ter uma maneira de verificar visualmente a confiança (por exemplo, confirmação de Transport Layer Security (TLS), mecanismos de website).
    - Para prevenir ataques [Machine-in-the-Middle (MITM)](0x04f-Testing-Network-Communication.md#intercepting-network-traffic-through-mitm), o cliente deve validar o nome de domínio totalmente qualificado do servidor com a chave pública que o servidor apresentou quando a conexão foi estabelecida.
- **Tipo de concessão:**
    - Em aplicativos nativos, a concessão de código deve ser usada em vez da concessão implícita.
    - Ao usar a concessão de código, o PKCE (Proof Key for Code Exchange) deve ser implementado para proteger a concessão de código. Certifique-se de que o servidor também o implemente.
    - O "código" de autenticação deve ser de curta duração e usado imediatamente após ser recebido. Verifique se os códigos de autenticação residem apenas na memória transitória e não são armazenados ou registrados.
- **Segredos do cliente:**
    - Segredos compartilhados não devem ser usados para provar a identidade do cliente porque o cliente poderia ser impersonado ("client_id" já serve como prova). Se eles usarem segredos do cliente, certifique-se de que eles estão armazenados em armazenamento local seguro.
- **Credenciais do usuário final:**
    - Proteja a transmissão de credenciais do usuário final com um método de camada de transporte, como TLS.
- **Tokens:**
    - Mantenha tokens de acesso na memória transitória.
    - Tokens de acesso devem ser transmitidos sobre uma conexão criptografada.
    - Reduza o escopo e a duração dos tokens de acesso quando a confidencialidade ponto a ponto não puder ser garantida ou o token fornecer acesso a informações ou transações sensíveis.
    - Lembre-se de que um atacante que roubou tokens pode acessar seu escopo e todos os recursos associados a eles se o aplicativo usar tokens de acesso como tokens bearer sem outra maneira de identificar o cliente.
    - Armazene tokens de atualização em armazenamento local seguro; eles são credenciais de longo prazo.

## Logout do Usuário

Falhar em destruir a sessão do lado do servidor é um dos erros de implementação de funcionalidade de logout mais comuns. Este erro mantém a sessão ou token vivo, mesmo após o usuário fazer logout do aplicativo. Um atacante que obtém informações de autenticação válidas pode continuar a usá-las e sequestrar a conta de um usuário.

Muitos aplicativos móveis não fazem logout automático dos usuários. Pode haver várias razões, como: porque é inconveniente para os clientes, ou devido a decisões tomadas ao implementar autenticação stateless. O aplicativo ainda deve ter uma função de logout, e deve ser implementada de acordo com as melhores práticas, destruindo todos os tokens ou identificadores de sessão armazenados localmente.

Se informações de sessão são armazenadas no servidor, elas devem ser destruídas enviando uma solicitação de logout para esse servidor. No caso de um aplicativo de alto risco, os tokens devem ser invalidados. Não remover tokens ou identificadores de sessão pode resultar em acesso não autorizado ao aplicativo no caso de os tokens serem vazados.
Note que outros tipos sensíveis de informação também devem ser removidos, pois qualquer informação que não seja adequadamente limpa pode ser vazada posteriormente, por exemplo durante um backup do dispositivo.

Aqui estão diferentes exemplos de término de sessão para logout adequado do lado do servidor:

- [Spring (Java)](https://docs.spring.io/autorepo/docs/spring-security/4.1.x/apidocs/org/springframework/security/web/authentication/logout/SecurityContextLogoutHandler.html "Spring (Java)")
- [Ruby on Rails](https://guides.rubyonrails.org/security.html "Ruby on Rails")
- [PHP](https://php.net/manual/en/function.session-destroy.php "PHP")

Se tokens de acesso e atualização são usados com autenticação stateless, eles devem ser excluídos do dispositivo móvel. O [token de atualização deve ser invalidado no servidor](https://auth0.com/blog/denylist-json-web-token-api-keys/ "Invalidating JSON Web Token API Keys").

O OWASP Web Testing Guide ([WSTG-SESS-06](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/06-Session_Management_Testing/06-Testing_for_Logout_Functionality "WSTG-SESS-006")) inclui uma explicação detalhada e mais casos de teste.

## Autenticação Suplementar

Esquemas de autenticação são às vezes suplementados por [autenticação contextual passiva](https://pdfs.semanticscholar.org/13aa/7bf53070ac8e209a84f6389bab58a1e2c888.pdf "Best Practices for
Multi-factor Authentication"), que pode incorporar:

- Geolocalização
- Endereço IP
- Hora do dia
- O dispositivo sendo usado

Idealmente, em tal sistema o contexto do usuário é comparado a dados previamente registrados para identificar anomalias que possam indicar abuso de conta ou potencial fraude. Este processo é transparente para o usuário, mas pode se tornar um poderoso impedimento para atacantes.

## Autenticação de Dois Fatores

Autenticação de dois fatores (2FA) é padrão para aplicativos que permitem que usuários acessem funções e dados sensíveis. Implementações comuns usam uma senha para o primeiro fator e qualquer um dos seguintes como o segundo fator:

- Senha única via SMS (SMS-OTP)
- Código único via chamada telefônica
- Token de hardware ou software
- Notificações push em combinação com PKI e autenticação local

Qualquer opção que seja usada, sempre deve ser aplicada e verificada no lado do servidor e nunca no lado do cliente. Caso contrário, o 2FA pode ser facilmente contornado dentro do aplicativo.

O 2FA pode ser realizado no login ou posteriormente na sessão do usuário.

> Por exemplo, após fazer login em um aplicativo bancário com um nome de usuário e PIN, o usuário está autorizado a realizar tarefas não sensíveis. Uma vez que o usuário tenta executar uma transferência bancária, o segundo fator ("autenticação step-up") deve ser apresentado.

**Melhores Práticas:**

- Não implemente seu próprio 2FA: Existem vários mecanismos de autenticação de dois fatores disponíveis que podem variar desde bibliotecas de terceiros, uso de aplicativos externos até verificações auto implementadas pelos desenvolvedores.
- Use OTPs de curta duração: Um OTP deve ser válido apenas por uma certa quantidade de tempo (geralmente 30 segundos) e após digitar o OTP incorretamente várias vezes (geralmente 3 vezes) o OTP fornecido deve ser invalidado e o usuário deve ser redirecionado para a página inicial ou deslogado.
- Armazene tokens com segurança: Para prevenir esses tipos de ataques, o aplicativo deve sempre verificar algum tipo de token de usuário ou outra informação dinâmica relacionada ao usuário que foi previamente armazenada com segurança (por exemplo, no Keychain/KeyStore).

### SMS-OTP

Embora senhas únicas (OTP) enviadas via SMS sejam um segundo fator comum para autenticação de dois fatores, este método tem suas deficiências. Em 2016, o NIST sugeriu: "Devido ao risco de que mensagens SMS possam ser interceptadas ou redirecionadas, implementadores de novos sistemas DEVEM considerar cuidadosamente autenticadores alternativos.". Abaixo você encontrará uma lista de algumas ameaças relacionadas e sugestões para evitar ataques bem-sucedidos em SMS-OTP.

Ameaças:

- Interceptação Sem Fio: O adversário pode interceptar mensagens SMS abusando de femtocells e outras vulnerabilidades conhecidas na rede de telecomunicações.
- Trojans: Aplicativos maliciosos instalados com acesso a mensagens de texto podem encaminhar o OTP para outro número ou backend.
- Ataque SIM SWAP: Neste ataque, o adversário liga para a companhia telefônica, ou trabalha para ela, e tem o número da vítima movido para um cartão SIM pertencente ao adversário. Se bem-sucedido, o adversário pode ver as mensagens SMS que são enviadas para o número de telefone da vítima. Isso inclui as mensagens usadas na autenticação de dois fatores.
- Ataque de Encaminhamento de Código de Verificação: Este ataque de engenharia social depende da confiança que os usuários têm na empresa que fornece o OTP. Neste ataque, o usuário recebe um código e é posteriormente solicitado a retransmitir esse código usando os mesmos meios pelos quais recebeu a informação.
- Correio de Voz: Alguns esquemas de autenticação de dois fatores permitem que o OTP seja enviado através de uma chamada telefônica quando o SMS não é mais preferido ou disponível. Muitas dessas chamadas, se não atendidas, enviam a informação para o correio de voz. Se um atacante conseguisse acesso ao correio de voz, ele também poderia usar o OTP para obter acesso à conta de um usuário.

Você pode encontrar abaixo várias sugestões para reduzir a probabilidade de exploração ao usar SMS para OTP:

- **Mensagens**: Ao enviar um OTP via SMS, certifique-se de incluir uma mensagem que informe ao usuário 1) o que fazer se ele não solicitou o código 2) sua empresa nunca ligará ou enviará mensagens solicitando que ele retransmita sua senha ou código.
- **Canal Dedicado**: Ao usar o recurso de notificação push do SO (AP
no iOS e FCM no Android), OTPs podem ser enviados com segurança para um aplicativo registrado. Esta informação é, comparada ao SMS, não acessível por outros aplicativos. Alternativamente de um OTP, a notificação push poderia acionar um pop-up para aprovar o acesso solicitado.
- **Entropia**: Use autenticadores com alta entropia para tornar OTPs mais difíceis de quebrar ou adivinhar e use pelo menos 6 dígitos. Certifique-se de que os dígitos sejam separados em grupos menores caso as pessoas precisem lembrá-los para copiá-los para seu aplicativo.
- **Evite Correio de Voz**: Se um usuário preferir receber uma chamada telefônica, não deixe a informação do OTP como um correio de voz.

**Pesquisa SMS-OTP:**

- [#dmitrienko] Dmitrienko, Alexandra, et al. "On the (in) security of mobile two-factor authentication." International Conference on Financial Cryptography and Data Security. Springer, Berlin, Heidelberg, 2014.
- [#grassi] Grassi, Paul A., et al. Digital identity guidelines: Authentication and lifecycle management (DRAFT). No. Special Publication (NIST SP)-800-63B. 2016.
- [#grassi2] Grassi, Paul A., et al. Digital identity guidelines: Authentication and lifecycle management. No. Special Publication (NIST SP)-800-63B. 2017.
- [#konoth] Konoth, Radhesh Krishnan, Victor van der Veen, and Herbert Bos. "How anywhere computing just killed your phone-based two-factor authentication." International Conference on Financial Cryptography and Data Security. Springer, Berlin, Heidelberg, 2016.
- [#mulliner] Mulliner, Collin, et al. "SMS-based one-time passwords: attacks and defense." International Conference on Detection of Intrusions and Malware, and Vulnerability Assessment. Springer, Berlin, Heidelberg, 2013.
- [#siadati] Siadati, Hossein, et al. "Mind your SMSes: Mitigating social engineering in second factor authentication." Computers & Security 65 (2017): 14-28.
- [#siadati2] Siadati, Hossein, Toan Nguyen, and Nasir Memon. "Verification code forwarding attack (short paper)." International Conference on Passwords. Springer, Cham, 2015.

### Assinatura de Transação com Notificações Push e PKI

Outro mecanismo alternativo e forte para implementar um segundo fator é a assinatura de transação.

A assinatura de transação requer autenticação da aprovação do usuário de transações críticas. Criptografia assimétrica é a melhor maneira de implementar assinatura de transação. O aplicativo gerará um par de chaves pública/privada quando o usuário se cadastrar, então registra a chave pública no backend. A chave privada é armazenada com segurança no KeyStore (Android) ou KeyChain (iOS). Para autorizar uma transação, o backend envia ao aplicativo móvel uma notificação push contendo os dados da transação. O usuário é então solicitado a confirmar ou negar a transação. Após confirmação, o usuário é solicitado a desbloquear o Keychain (inserindo o PIN ou impressão digital), e os dados são assinados com a chave privada do usuário. A transação assinada é então enviada para o servidor, que verifica a assinatura com a chave pública do usuário.

## Atividade de Login e Bloqueio de Dispositivo

É uma melhor prática que aplicativos devam informar o usuário sobre todas as atividades de login dentro do aplicativo com a possibilidade de bloquear certos dispositivos. Isso pode ser dividido em vários cenários:

1. O aplicativo fornece uma notificação push no momento em que sua conta é usada em outro dispositivo para notificar o usuário de atividades diferentes. O usuário pode então bloquear este dispositivo após abrir o aplicativo via notificação push.
2. O aplicativo fornece uma visão geral da última sessão após o login. Se a sessão anterior foi com uma configuração diferente (por exemplo, localização, dispositivo, versão do aplicativo) comparada à configuração atual, então o usuário deve ter a opção de reportar atividades suspeitas e bloquear dispositivos usados na sessão anterior.
3. O aplicativo fornece uma visão geral da última sessão após o login em todos os momentos.
4. O aplicativo tem um portal de autoatendimento no qual o usuário pode ver um log de auditoria. Isso permite que o usuário gerencie os diferentes dispositivos que estão logados.

O desenvolvedor pode fazer uso de meta-informação específica e associá-la a cada atividade ou evento diferente dentro do aplicativo. Isso tornará mais fácil para o usuário detectar comportamento suspeito e bloquear o dispositivo correspondente. A meta-informação pode incluir:

- Dispositivo: O usuário pode identificar claramente todos os dispositivos onde o aplicativo está sendo usado.
- Data e Hora: O usuário pode ver claramente a última data e hora quando o aplicativo foi usado.
- Localização: O usuário pode identificar claramente os últimos locais onde o aplicativo foi usado.

O aplicativo pode fornecer uma lista de histórico de atividades que será atualizada após cada atividade sensível dentro do aplicativo. A escolha de quais atividades auditar precisa ser feita para cada aplicativo com base nos dados que ele manipula e no nível de risco de segurança que a equipe está disposta a ter. Abaixo está uma lista de atividades sensíveis comuns que são geralmente auditadas:

- Tentativas de login
- Alterações de senha
- Alterações de Informações Pessoais Identificáveis (nome, endereço de email, número de telefone, etc.)
- Atividades sensíveis (compra, acesso a recursos importantes, etc.)
- Consentimento a cláusulas de Termos e Condições

Conteúdo pago requer cuidado especial, e meta-informação adicional (por exemplo, custo da operação, crédito, etc.) pode ser usada para garantir o conhecimento do usuário sobre todos os parâmetros da operação.

Além disso, mecanismos de não-repúdio devem ser aplicados a transações sensíveis (por exemplo, acesso a conteúdo pago, consentimento dado a cláusulas de Termos e Condições, etc.) para provar que uma transação específica foi de fato realizada (integridade) e por quem (autenticação).

Por último, deve ser possível para o usuário fazer logout de sessões abertas específicas e em alguns casos pode ser interessante bloquear totalmente certos dispositivos usando um identificador de dispositivo.