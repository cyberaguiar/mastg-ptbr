
---
masvs_category: MASVS-CODE
platform: all
---

# Qualidade de Código em Aplicativos Móveis

Desenvolvedores de aplicativos móveis usam uma ampla variedade de linguagens de programação e frameworks. Como tal, vulnerabilidades comuns, como injeção de SQL, estouros de buffer e cross-site scripting (XSS), podem se manifestar em aplicativos quando práticas de programação seguras são negligenciadas.

As mesmas falhas de programação podem afetar aplicativos Android e iOS em algum grau, então forneceremos uma visão geral das classes de vulnerabilidade mais comuns frequentemente na seção geral do guia. Em seções posteriores, abordaremos instâncias específicas do sistema operacional e recursos de mitigação de exploração.

## Falhas de Injeção

Uma _falha de injeção_ descreve uma classe de vulnerabilidade de segurança que ocorre quando a entrada do usuário é inserida em consultas ou comandos de backend. Ao injetar metacaracteres, um atacante pode executar código malicioso que é inadvertidamente interpretado como parte do comando ou consulta. Por exemplo, manipulando uma consulta SQL, um atacante poderia recuperar registros arbitrários do banco de dados ou manipular o conteúdo do banco de dados backend.

Vulnerabilidades desta classe são mais prevalentes em serviços web do lado do servidor. Instâncias exploráveis também existem dentro de aplicativos móveis, mas as ocorrências são menos comuns, além de a superfície de ataque ser menor.

Por exemplo, enquanto um aplicativo pode consultar um banco de dados SQLite local, esses bancos de dados geralmente não armazenam dados sensíveis (assumindo que o desenvolvedor seguiu práticas básicas de segurança). Isso torna a injeção de SQL um vetor de ataque não viável. No entanto, vulnerabilidades de injeção exploráveis às vezes ocorrem, o que significa que a validação adequada de entrada é uma prática necessária para programadores.

### Injeção de SQL

Um ataque de _injeção de SQL_ envolve integrar comandos SQL em dados de entrada, imitando a sintaxe de um comando SQL predefinido. Um ataque de injeção de SQL bem-sucedido permite que o atacante leia ou escreva no banco de dados e possivelmente execute comandos administrativos, dependendo das permissões concedidas pelo servidor.

Aplicativos em Android e iOS usam bancos de dados SQLite como meio de controlar e organizar o armazenamento local de dados. Suponha que um aplicativo Android manipule a autenticação local do usuário armazenando as credenciais do usuário em um banco de dados local (uma prática de programação pobre que ignoraremos para este exemplo). No login, o aplicativo consulta o banco de dados para procurar um registro com o nome de usuário e senha inseridos pelo usuário:

```java
SQLiteDatabase db;

String sql = "SELECT * FROM users WHERE username = '" +  username + "' AND password = '" + password +"'";

Cursor c = db.rawQuery( sql, null );

return c.getCount() != 0;
```

Vamos supor ainda que um atacante insira os seguintes valores nos campos "username" e "password":

```sql
username = 1' or '1' = '1
password = 1' or '1' = '1
```

Isso resulta na seguinte consulta:

```sql
SELECT * FROM users WHERE username='1' OR '1' = '1' AND Password='1' OR '1' = '1'
```

Como a condição `'1' = '1'` sempre avalia como verdadeira, esta consulta retorna todos os registros no banco de dados, fazendo com que a função de login retorne `true` mesmo que nenhuma conta de usuário válida tenha sido inserida.

O Ostorlab explorou o parâmetro sort do [aplicativo móvel de clima do Yahoo](https://blog.ostorlab.co/android-sql-contentProvider-sql-injections.html) com adb usando esta carga útil de injeção de SQL.

Outra instância real de injeção de SQL do lado do cliente foi descoberta por Mark Woods dentro dos aplicativos Android "Qnotes" e "Qget" em execução em dispositivos de armazenamento QNAP NAS. Esses aplicativos exportaram provedores de conteúdo vulneráveis à injeção de SQL, permitindo que um atacante recuperasse as credenciais para o dispositivo NAS. Uma descrição detalhada deste problema pode ser encontrada no [Blog da Nettitude](https://blog.nettitude.com/uk/qnap-android-dont-provide "Nettitude Blog - QNAP Android: Don't Over Provide").

### Injeção de XML

Em um ataque de _injeção de XML_, o atacante injeta metacaracteres XML para alterar estruturalmente o conteúdo XML. Isso pode ser usado para comprometer a lógica de um aplicativo ou serviço baseado em XML, bem como possivelmente permitir que um atacante explore a operação do analisador XML que processa o conteúdo.

Uma variante popular deste ataque é [XML eXternal Entity (XXE)](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_%28XXE%29_Processing "Ataque de Entidade Externa XML (XXE)"). Aqui, um atacante injeta uma definição de entidade externa contendo um URI no XML de entrada. Durante a análise, o analisador XML expande a entidade definida pelo atacante acessando o recurso especificado pelo URI. A integridade do aplicativo de análise determina em última instância as capacidades concedidas ao atacante, onde o usuário malicioso pode fazer qualquer (ou todos) dos seguintes: acessar arquivos locais, acionar solicitações HTTP para hosts e portas arbitrárias, lançar um ataque [cross-site request forgery (CSRF)](https://owasp.org/www-community/attacks/csrf "Cross-Site Request Forgery (CSRF)") e causar uma condição de negação de serviço. O guia de teste web da OWASP contém o [seguinte exemplo para XXE](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/07-Testing_for_XML_Injection "Testando para Injeção XML"):

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
 <!DOCTYPE foo [
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///dev/random" >]><foo>&xxe;</foo>
```

Neste exemplo, o arquivo local `/dev/random` é aberto onde um fluxo interminável de bytes é retornado, potencialmente causando uma negação de serviço.

A tendência atual no desenvolvimento de aplicativos concentra-se principalmente em serviços baseados em REST/JSON, pois o XML está se tornando menos comum. No entanto, nos casos raros em que conteúdo fornecido pelo usuário ou não confiável é usado para construir consultas XML, ele pode ser interpretado por analisadores XML locais, como NSXMLParser no iOS. Como tal, essa entrada deve sempre ser validada e os metacaracteres devem ser escapados.

### Vetores de Ataque de Injeção

A superfície de ataque de aplicativos móveis é bastante diferente de aplicações web e de rede típicas. Aplicativos móveis não costumam expor serviços na rede, e vetores de ataque viáveis na interface do usuário de um aplicativo são raros. Ataques de injeção contra um aplicativo são mais prováveis de ocorrer através de interfaces de comunicação interprocessos (IPC), onde um aplicativo malicioso ataca outro aplicativo em execução no dispositivo.

Localizar uma vulnerabilidade potencial começa por:

- Identificar possíveis pontos de entrada para entrada não confiável e, em seguida, rastrear desses locais para ver se o destino contém funções potencialmente vulneráveis.
- Identificar chamadas de biblioteca/API perigosas conhecidas (por exemplo, consultas SQL) e, em seguida, verificar se a entrada não verificada interface com sucesso com as respectivas consultas.

Durante uma revisão de segurança manual, você deve empregar uma combinação de ambas as técnicas. Em geral, entradas não confiáveis entram em aplicativos móveis através dos seguintes canais:

- Chamadas IPC
- Esquemas de URL personalizados
- Códigos QR
- Arquivos de entrada recebidos via Bluetooth, NFC ou outros meios
- Pasteboards
- Interface do usuário

Verifique se as seguintes melhores práticas foram seguidas:

- Entradas não confiáveis são verificadas por tipo e/ou validadas usando uma lista de valores aceitáveis.
- Instruções preparadas com vinculação de variáveis (ou seja, consultas parametrizadas) são usadas ao realizar consultas de banco de dados. Se instruções preparadas são definidas, dados fornecidos pelo usuário e código SQL são automaticamente separados.
- Ao analisar dados XML, certifique-se de que o aplicativo do analisador está configurado para rejeitar a resolução de entidades externas para prevenir ataques XXE.
- Ao trabalhar com dados de certificado no formato x509, certifique-se de que analisadores seguros são usados. Por exemplo, Bouncy Castle abaixo da versão 1.6 permite Execução Remota de Código por meio de reflexão insegura.

Cobriremos detalhes relacionados a fontes de entrada e APIs potencialmente vulneráveis para cada sistema operacional móvel nos guias de teste específicos do sistema operacional.

## Falhas de Cross-Site Scripting

Problemas de cross-site scripting (XSS) permitem que atacantes injetem scripts do lado do cliente em páginas web visualizadas por usuários. Este tipo de vulnerabilidade é prevalente em aplicações web. Quando um usuário visualiza o script injetado em um navegador, o atacante ganha a capacidade de contornar a política de mesma origem, permitindo uma ampla variedade de explorações (por exemplo, roubar cookies de sessão, registrar pressionamentos de tecla, realizar ações arbitrárias, etc.).

No contexto de _aplicativos nativos_, os riscos de XSS são muito menos prevalentes pela simples razão de que esses tipos de aplicações não dependem de um navegador web. No entanto, aplicativos que usam componentes WebView, como `WKWebView` ou o obsoleto `UIWebView` no iOS e `WebView` no Android, são potencialmente vulneráveis a tais ataques.

Um exemplo mais antigo, mas bem conhecido, é o [problema de XSS local no aplicativo Skype para iOS, identificado pela primeira vez por Phil Purviance](https://superevr.com/blog/2011/xss-in-skype-for-ios "XSS no Skype para iOS"). O aplicativo Skype não conseguiu codificar adequadamente o nome do remetente da mensagem, permitindo que um atacante injetasse JavaScript malicioso para ser executado quando um usuário visualiza a mensagem. Nesta prova de conceito, Phil mostrou como explorar o problema e roubar a lista de contatos de um usuário.

### Análise Estática - Considerações de Teste de Segurança

Observe atentamente quaisquer WebViews presentes e investigue se há entrada não confiável renderizada pelo aplicativo.

Problemas de XSS podem existir se a URL aberta pelo WebView for parcialmente determinada pela entrada do usuário. O seguinte exemplo é de um problema de XSS no [Serviço Web Zoho, relatado por Linus Särud](https://labs.detectify.com/2015/02/20/finding-an-xss-in-an-html-based-android-application/ "Encontrando um XSS em um aplicativo Android baseado em HTML").

Java

```java
webView.loadUrl("javascript:initialize(" + myNumber + ");");
```

Kotlin

```kotlin
webView.loadUrl("javascript:initialize($myNumber);")
```

Outro exemplo de problemas de XSS determinados pela entrada do usuário são métodos públicos sobrescritos.

Java

```java
@Override
public boolean shouldOverrideUrlLoading(WebView view, String url) {
  if (url.substring(0,6).equalsIgnoreCase("yourscheme:")) {
    // parse the URL object and execute functions
  }
}
```

Kotlin

```kotlin
    fun shouldOverrideUrlLoading(view: WebView, url: String): Boolean {
        if (url.substring(0, 6).equals("yourscheme:", ignoreCase = true)) {
            // parse the URL object and execute functions
        }
    }
```

Sergey Bobrov foi capaz de tirar vantagem disso no seguinte [relatório do HackerOne](https://hackerone.com/reports/189793 "Relatório do HackerOne - [Android] XSS via start ContentActivity"). Qualquer entrada para o parâmetro HTML seria confiada no ActionBarContentActivity do Quora. Cargas úteis foram bem-sucedidas usando adb, dados da área de transferência via ModalContentActivity e Intents de aplicativos de terceiros.

- ADB

  ```bash
  $ adb shell
  $ am start -n com.quora.android/com.quora.android.ActionBarContentActivity \
  -e url 'http://test/test' -e html 'XSS<script>alert(123)</script>'
  ```

- Dados da Área de Transferência

  ```bash
  $ am start -n com.quora.android/com.quora.android.ModalContentActivity  \
  -e url 'http://test/test' -e html \
  '<script>alert(QuoraAndroid.getClipboardData());</script>'
  ```

- Intent de terceiros em Java ou Kotlin:

  ```java
  Intent i = new Intent();
  i.setComponent(new ComponentName("com.quora.android",
  "com.quora.android.ActionBarContentActivity"));
  i.putExtra("url","http://test/test");
  i.putExtra("html","XSS PoC <script>alert(123)</script>");
  view.getContext().startActivity(i);
  ```

  ```kotlin
  val i = Intent()
  i.component = ComponentName("com.quora.android",
  "com.quora.android.ActionBarContentActivity")
  i.putExtra("url", "http://test/test")
  i.putExtra("html", "XSS PoC <script>alert(123)</script>")
  view.context.startActivity(i)
  ```

Se um WebView for usado para exibir um site remoto, o ônus de escapar HTML recai sobre o lado do servidor. Se existir uma falha de XSS no servidor web, isso pode ser usado para executar script no contexto do WebView. Como tal, é importante realizar análise estática do código-fonte da aplicação web.

Verifique se as seguintes melhores práticas foram seguidas:

- Nenhum dado não confiável é renderizado em HTML, JavaScript ou outros contextos interpretados, a menos que seja absolutamente necessário.
- Codificação apropriada é aplicada para escapar caracteres, como codificação de entidade HTML. Nota: regras de escape tornam-se complicadas quando HTML está aninhado dentro de outro código, por exemplo, renderizando uma URL localizada dentro de um bloco JavaScript.

Considere como os dados serão renderizados em uma resposta. Por exemplo, se os dados são renderizados em um contexto HTML, seis caracteres de controle que devem ser escapados:

| Caractere  | Escapado      |
| :-------------: |:-------------:|
| & | &amp;|
| < | &lt; |
| > | &gt;|
| " | &quot;|
| ' | &#x27;|
| / | &#x2F;|

Para uma lista abrangente de regras de escape e outras medidas de prevenção, consulte a [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html "OWASP XSS Prevention Cheat Sheet").

### Análise Dinâmica - Considerações de Teste de Segurança

Problemas de XSS podem ser melhor detectados usando fuzzing de entrada manual e/ou automatizado, ou seja, injetando tags HTML e caracteres especiais em todos os campos de entrada disponíveis para verificar se a aplicação web nega entradas inválidas ou escapa os metacaracteres HTML em sua saída.

Um [ataque de XSS refletido](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/01-Testing_for_Reflected_Cross_Site_Scripting.html "Testando para Cross site scripting refletido") refere-se a uma exploração onde código malicioso é injetado via um link malicioso. Para testar esses ataques, o fuzzing de entrada automatizado é considerado um método eficaz. Por exemplo, o [BURP Scanner](https://portswigger.net/burp/ "Burp Suite") é altamente eficaz na identificação de vulnerabilidades de XSS refletido. Como sempre com análise automatizada, certifique-se de que todos os vetores de entrada sejam cobertos com uma revisão manual dos parâmetros de teste.

## Bugs de Corrupção de Memória

Bugs de corrupção de memória são um ponto forte popular entre hackers. Esta

classe de bug resulta de um erro de programação que faz com que o programa acesse um local de memória não intencional. Sob as condições certas, os atacantes podem capitalizar esse comportamento para sequestrar o fluxo de execução do programa vulnerável e executar código arbitrário. Esse tipo de vulnerabilidade ocorre de várias maneiras:

- **Estouros de buffer**: Isso descreve um erro de programação onde um aplicativo escreve além de um intervalo de memória alocado para uma operação particular. Um atacante pode usar essa falha para sobrescrever dados de controle importantes localizados na memória adjacente, como ponteiros de função. Estouros de buffer eram anteriormente o tipo mais comum de falha de corrupção de memória, mas tornaram-se menos prevalentes ao longo dos anos devido a vários fatores. Notavelmente, a conscientização entre desenvolvedores sobre os riscos de usar funções inseguras da biblioteca C é agora uma prática comum, além disso, capturar bugs de estouro de buffer é relativamente simples. No entanto, ainda vale a pena testar tais defeitos.

- **Acesso fora dos limites**: Aritmética de ponteiro com bugs pode fazer com que um ponteiro ou índice referencie uma posição além dos limites da estrutura de memória pretendida (por exemplo, buffer ou lista). Quando um aplicativo tenta escrever em um endereço fora dos limites, ocorre uma falha ou comportamento não intencional. Se o atacante puder controlar o deslocamento de destino e manipular o conteúdo escrito até certo ponto, [a exploração de execução de código é provavelmente possível](https://www.zerodayinitiative.com/advisories/ZDI-17-110/ "Exemplo do Adobe Flash Mediaplayer").

- **Ponteiros pendurados**: Estes ocorrem quando um objeto com uma referência de entrada para um local de memória é excluído ou desalocado, mas o ponteiro do objeto não é redefinido. Se o programa posteriormente usar o ponteiro _pendurado_ para chamar uma função virtual do objeto já desalocado, é possível sequestrar a execução sobrescrevendo o ponteiro vtable original. Alternativamente, é possível ler ou escrever variáveis de objeto ou outras estruturas de memória referenciadas por um ponteiro pendurado.

- **Use-after-free**: Isso se refere a um caso especial de ponteiros pendurados referenciando memória liberada (desalocada). Depois que um endereço de memória é limpo, todos os ponteiros que referenciam o local se tornam inválidos, fazendo com que o gerenciador de memória retorne o endereço para um pool de memória disponível. Quando este local de memória é eventualmente realocado, acessar o ponteiro original lerá ou escreverá os dados contidos na memória recém-alocada. Isso geralmente leva à corrupção de dados e comportamento indefinido, mas atacantes astutos podem configurar os locais de memória apropriados para alavancar o controle do ponteiro de instrução.

- **Estouros de inteiro**: Quando o resultado de uma operação aritmética excede o valor máximo para o tipo inteiro definido pelo programador, isso resulta no valor "envolvendo" o valor máximo do inteiro, inevitavelmente resultando em um valor pequeno sendo armazenado. Por outro lado, quando o resultado de uma operação aritmética é menor que o valor mínimo do tipo inteiro, ocorre um _estouro negativo de inteiro_ (integer underflow) onde o resultado é maior que o esperado. Se um bug particular de estouro/estouro negativo de inteiro é explorável depende de como o inteiro é usado. Por exemplo, se o tipo inteiro representasse o comprimento de um buffer, isso poderia criar uma vulnerabilidade de estouro de buffer.

- **Vulnerabilidades de string de formato**: Quando a entrada não verificada do usuário é passada para o parâmetro de string de formato das funções da família `printf` do C, os atacantes podem injetar tokens de formato como '%c' e '%n' para acessar a memória. Bugs de string de formato são convenientes de explorar devido à sua flexibilidade. Se um programa produzir o resultado da operação de formatação de string, o atacante pode ler e escrever na memória arbitrariamente, contornando assim recursos de proteção como ASLR.

O objetivo principal na exploração de corrupção de memória é geralmente redirecionar o fluxo do programa para um local onde o atacante colocou instruções de máquina montadas referidas como _shellcode_. No iOS, o recurso de prevenção de execução de dados (como o nome implica) impede a execução de memória definida como segmentos de dados. Para contornar essa proteção, os atacantes alavancam a programação orientada a retorno (ROP). Este processo envolve encadear pequenos pedaços de código pré-existentes ("gadgets") no segmento de texto onde esses gadgets podem executar uma função útil para o atacante ou chamar `mprotect` para alterar as configurações de proteção de memória para o local onde o atacante armazenou o _shellcode_.

Aplicativos Android são, em sua maioria, implementados em Java, que é inerentemente seguro contra problemas de corrupção de memória por design. No entanto, aplicativos nativos utilizando bibliotecas JNI são suscetíveis a esse tipo de bug. Em casos raros, aplicativos Android que usam analisadores XML/JSON para desembrulhar objetos Java também estão sujeitos a bugs de corrupção de memória. [Um exemplo](https://blog.oversecured.com/Exploiting-memory-corruption-vulnerabilities-on-Android/#example-of-the-vulnerability-in-paypal%E2%80%99s-apps) de tal vulnerabilidade foi encontrado no aplicativo PayPal.

Da mesma forma, aplicativos iOS podem envolver chamadas C/C++ em Obj-C ou Swift, tornando-os suscetíveis a esse tipo de ataque.

**Exemplo:**

O seguinte snippet de código mostra um exemplo simples para uma condição resultando em uma vulnerabilidade de estouro de buffer.

```c
 void copyData(char *userId) {
    char  smallBuffer[10]; // tamanho de 10
    strcpy(smallBuffer, userId);
 }
```

Para identificar possíveis estouros de buffer, procure por usos de funções de string inseguras (`strcpy`, `strcat`, outras funções começando com o prefixo "str", etc.) e construções de programação potencialmente vulneráveis, como copiar entrada do usuário em um buffer de tamanho limitado. O seguinte deve ser considerado bandeiras vermelhas para funções de string inseguras:

- `strcat`
- `strcpy`
- `strncat`
- `strlcat`
- `strncpy`
- `strlcpy`
- `sprintf`
- `snprintf`
- `gets`

Além disso, procure por instâncias de operações de cópia implementadas como loops "for" ou "while" e verifique se as verificações de comprimento são realizadas corretamente.

Verifique se as seguintes melhores práticas foram seguidas:

- Ao usar variáveis inteiras para indexação de array, cálculos de comprimento de buffer ou qualquer outra operação crítica para segurança, verifique se tipos inteiros não assinados são usados e testes de pré-condição são realizados para prevenir a possibilidade de envolvimento de inteiro.
- O aplicativo não usa funções de string inseguras como `strcpy`, a maioria das outras funções começando com o prefixo "str", `sprint`, `vsprintf`, `gets`, etc.;
- Se o aplicativo contiver código C++, classes de string ANSI C++ são usadas;
- No caso de `memcpy`, certifique-se de verificar se o buffer de destino é pelo menos do mesmo tamanho que a fonte e que ambos os buffers não estão se sobrepondo.
- Aplicativos iOS escritos em Objective-C usam a classe NSString. Aplicativos C no iOS devem usar CFString, a representação Core Foundation de uma string.
- Nenhum dado não confiável é concatenado em strings de formato.

### Considerações de Teste de Segurança de Análise Estática

Análise estática de código de baixo nível é um tópico complexo que poderia facilmente preencher seu próprio livro. Ferramentas automatizadas como [RATS](https://code.google.com/archive/p/rough-auditing-tool-for-security/downloads "RATS - Ferramenta de auditoria aproximada para segurança") combinadas com esforços limitados de inspeção manual são geralmente suficientes para identificar frutas ao alcance da mão. No entanto, condições de corrupção de memória frequentemente decorrem de causas complexas. Por exemplo, um bug use-after-free pode realmente ser o resultado de uma condição de corrida intrincada e contra-intuitiva não imediatamente aparente. Bugs que se manifestam a partir de instâncias profundas de deficiências de código negligenciadas são geralmente descobertos através de análise dinâmica ou por testadores que investem tempo para obter um entendimento profundo do programa.

### Considerações de Teste de Segurança de Análise Dinâmica

Bugs de corrupção de memória são melhor descobertos via fuzzing de entrada: uma técnica de teste de software de caixa preta automatizada na qual dados malformados são continuamente enviados para um aplicativo para pesquisar por condições de vulnerabilidade potencial. Durante este processo, o aplicativo é monitorado por mau funcionamento e falhas. Se ocorrer uma falha, a esperança (pelo menos para testadores de segurança) é que as condições criando a falha revelem uma falha de segurança explorável.

Técnicas ou scripts de teste de fuzzing (frequentemente chamados de "fuzzers") normalmente gerarão múltiplas instâncias de entrada estruturada de uma forma semi-correta. Essencialmente, os valores ou argumentos gerados são pelo menos parcialmente aceitos pelo aplicativo alvo, mas também contêm elementos inválidos, potencialmente acionando falhas de processamento de entrada e comportamentos inesperados do programa. Um bom fuzzer expõe uma quantidade substancial de possíveis caminhos de execução do programa (ou seja, saída de alta cobertura). As entradas são geradas do zero ("baseado em geração") ou derivadas da mutação de dados de entrada válidos conhecidos ("baseado em mutação").

Para mais informações sobre fuzzing, consulte o [OWASP Fuzzing Guide](https://owasp.org/www-community/Fuzzing "OWASP Fuzzing Guide").

## Mecanismos de Proteção Binária

### Código Independente de Posição

[PIC (Position Independent Code)](https://en.wikipedia.org/wiki/Position-independent_code) é código que, sendo colocado em algum lugar na memória principal, executa adequadamente independentemente de seu endereço absoluto. PIC é comumente usado para bibliotecas compartilhadas, para que o mesmo código de biblioteca possa ser carregado em um local em cada espaço de endereço do programa onde não se sobreponha com outra memória em uso (por exemplo, outras bibliotecas compartilhadas).

PIE (Position Independent Executable) são binários executáveis feitos inteiramente de PIC. Binários PIE são usados para habilitar [ASLR (Address Space Layout Randomization)](https://en.wikipedia.org/wiki/Address_space_layout_randomization) que organiza aleatoriamente as posições de espaço de endereço de áreas de dados chave de um processo, incluindo a base do executável e as posições da pilha, heap e bibliotecas.

### Gerenciamento de Memória

#### Contagem Automática de Referência

[ARC (Automatic Reference Counting)](https://en.wikipedia.org/wiki/Automatic_Reference_Counting) é um recurso de gerenciamento de memória do compilador Clang exclusivo para [Objective-C](https://developer.apple.com/library/content/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html) e [Swift](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html). ARC libera automaticamente a memória usada por instâncias de classe quando essas instâncias não são mais necessárias. ARC difere da coleta de lixo por rastreamento
que não há um processo em segundo plano que desaloque os objetos de forma assíncrona em tempo de execução.

Ao contrário da coleta de lixo por rastreamento, o ARC não trata ciclos de referência automaticamente. Isso significa que, enquanto houver referências "fortes" para um objeto, ele não será desalocado. Referências cruzadas fortes podem, portanto, criar deadlocks e vazamentos de memória. Cabe ao desenvolvedor quebrar ciclos usando referências fracas. Você pode aprender mais sobre como isso difere da Coleta de Lixo [aqui](https://fragmentedpodcast.com/episodes/064/).

#### Coleta de Lixo

[Coleta de Lixo (GC)](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)) é um recurso de gerenciamento de memória automático de algumas linguagens como Java/Kotlin/Dart. O coletor de lixo tenta recuperar memória que foi alocada pelo programa, mas não é mais referenciada - também chamada de lixo. O Android runtime (ART) faz uso de uma [versão melhorada de GC](https://source.android.com/devices/tech/dalvik#Improved_GC). Você pode aprender mais sobre como isso difere do ARC [aqui](https://fragmentedpodcast.com/episodes/064/).

#### Gerenciamento Manual de Memória

[Gerenciamento manual de memória](https://en.wikipedia.org/wiki/Manual_memory_management) é tipicamente necessário em bibliotecas nativas escritas em C/C++ onde ARC e GC não se aplicam. O desenvolvedor é responsável por fazer o gerenciamento adequado de memória. O gerenciamento manual de memória é conhecido por habilitar várias classes principais de bugs em um programa quando usado incorretamente, notavelmente violações de [segurança de memória](https://en.wikipedia.org/wiki/Memory_safety) ou [vazamentos de memória](https://en.wikipedia.org/wiki/Memory_leak).

Mais informações podem ser encontradas em ["Bugs de Corrupção de Memória"](#memory-corruption-bugs).

### Proteção contra Stack Smashing

[Stack canaries](https://en.wikipedia.org/wiki/Stack_buffer_overflow#Stack_canaries) ajudam a prevenir ataques de estouro de buffer de pilha armazenando um valor inteiro oculto na pilha logo antes do ponteiro de retorno. Este valor é então validado antes da instrução de retorno da função ser executada. Um ataque de estouro de buffer frequentemente sobrescreve uma região de memória para sobrescrever o ponteiro de retorno e assumir o controle do fluxo do programa. Se os stack canaries estiverem habilitados, eles também serão sobrescritos e a CPU saberá que a memória foi adulterada.

Estouro de buffer de pilha é um tipo da vulnerabilidade de programação mais geral conhecida como [estouro de buffer](https://en.wikipedia.org/wiki/Buffer_overflow) (ou transbordamento de buffer). Preencher excessivamente um buffer na pilha é mais provável de **desviar a execução do programa** do que preencher excessivamente um buffer no heap porque a pilha contém os endereços de retorno para todas as chamadas de função ativas.