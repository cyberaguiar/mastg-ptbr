
---
masvs_category: MASVS-RESILIENCE
platform: all
---

# Adulteração e Engenharia Reversa de Aplicativos Móveis

As técnicas de engenharia reversa e adulteração há muito pertencem ao domínio de crackers, modders, analistas de malware, etc. Para testadores e pesquisadores de segurança "tradicionais", a engenharia reversa tem sido mais uma habilidade complementar. Mas as marés estão mudando: o teste de caixa preta de aplicativos móveis requer cada vez mais a desmontagem de aplicativos compilados, aplicação de patches e adulteração de código binário ou até mesmo de processos em execução. O fato de que muitos aplicativos móveis implementam defesas contra adulteração indesejada não facilita as coisas para os testadores de segurança.

A engenharia reversa de um aplicativo móvel é o processo de analisar o aplicativo compilado para extrair informações sobre seu código-fonte. O objetivo da engenharia reversa é _compreender_ o código.

A _adulteração_ é o processo de alterar um aplicativo móvel (seja o aplicativo compilado ou o processo em execução) ou seu ambiente para afetar seu comportamento. Por exemplo, um aplicativo pode se recusar a ser executado em seu dispositivo de teste com root, tornando impossível executar alguns de seus testes. Nesses casos, você vai querer alterar o comportamento do aplicativo.

Os testadores de segurança móvel são bem servidos ao entender conceitos básicos de engenharia reversa. Eles também devem conhecer dispositivos móveis e sistemas operacionais por completo: arquitetura do processador, formato executável, intricidades da linguagem de programação e assim por diante.

A engenharia reversa é uma arte, e descrever cada faceta dela preencheria uma biblioteca inteira. A enorme variedade de técnicas e especializações é impressionante: pode-se passar anos trabalhando em um subproblema muito específico e isolado, como automatizar a análise de malware ou desenvolver novos métodos de desofuscação. Os testadores de segurança são generalistas; para serem engenheiros reversos eficazes, eles devem filtrar a vasta quantidade de informações relevantes.

Não existe um processo genérico de engenharia reversa que sempre funcione. Dito isso, descreveremos métodos e ferramentas comumente usados posteriormente neste guia e daremos exemplos de como lidar com as defesas mais comuns.

## Por que você precisa disso

O teste de segurança móvel requer pelo menos habilidades básicas de engenharia reversa por várias razões:

**1. Para permitir o teste de caixa preta de aplicativos móveis.** Aplicativos modernos geralmente incluem controles que dificultarão a análise dinâmica. O SSL pinning e a criptografia ponto a ponto (E2E) às vezes impedem você de interceptar ou manipular o tráfego com um proxy. A detecção de root pode impedir que o aplicativo seja executado em um dispositivo com root, impedindo você de usar ferramentas avançadas de teste. Você deve ser capaz de desativar essas defesas.

**2. Para melhorar a análise estática no teste de segurança de caixa preta.** Em um teste de caixa preta, a análise estática do bytecode ou código binário do aplicativo ajuda você a entender a lógica interna do aplicativo. Também permite identificar falhas como credenciais embutidas no código.

**3. Para avaliar a resiliência contra engenharia reversa.** Aplicativos que implementam as medidas de proteção de software listadas nos Controles Anti-Reversão do Padrão de Verificação de Segurança de Aplicativos Móveis (MASVS-R) devem resistir à engenharia reversa até certo ponto. Para verificar a eficácia de tais controles, o testador pode realizar uma _avaliação de resiliência_ como parte do teste de segurança geral. Para a avaliação de resiliência, o testador assume o papel do engenheiro reverso e tenta contornar as defesas.

Antes de mergulharmos no mundo da reversão de aplicativos móveis, temos algumas boas e más notícias. Vamos começar com as boas notícias:

**No final, o engenheiro reverso sempre vence.**

Isso é particularmente verdadeiro na indústria móvel, onde o engenheiro reverso tem uma vantagem natural: a forma como os aplicativos móveis são implantados e sandboxados é por design mais restritiva do que a implantação e sandboxing de aplicativos clássicos de desktop, então incluir os mecanismos defensivos semelhantes a rootkits frequentemente encontrados em software Windows (por exemplo, sistemas DRM) simplesmente não é viável. A abertura do Android permite que engenheiros reversos façam alterações favoráveis ao sistema operacional, auxiliando o processo de engenharia reversa. O iOS dá aos engenheiros reversos menos controle, mas as opções defensivas também são mais limitadas.

A má notícia é que lidar com controles anti-depuração multi-threaded, white-boxes criptográficos, recursos anti-adulteração furtivos e transformações de fluxo de controle altamente complexas não é para os fracos de coração. Os esquemas de proteção de software mais eficazes são proprietários e não serão vencidos com ajustes e truques padrão. Derrotá-los requer análise manual tediosa, codificação, frustração e, dependendo da sua personalidade, noites sem dormir e relacionamentos tensos.

É fácil para iniciantes ficarem sobrecarregados com o enorme escopo da reversão. A melhor maneira de começar é configurar algumas ferramentas básicas (veja as seções relevantes nos capítulos de reversão do Android e iOS) e começar com tarefas simples de reversão e crackmes. Você precisará aprender sobre a linguagem assembly/bytecode, o sistema operacional, ofuscações que encontrar e assim por diante. Comece com tarefas simples e gradualmente avance para as mais difíceis.

Na seção a seguir, daremos uma visão geral das técnicas mais comumente usadas no teste de segurança de aplicativos móveis. Em capítulos posteriores, nos aprofundaremos em detalhes específicos do sistema operacional, tanto do Android quanto do iOS.

## Técnicas Básicas de Adulteração

### Aplicação de Patches Binários

A _aplicação de patches_ é o processo de alterar o aplicativo compilado, por exemplo, alterando código em executáveis binários, modificando bytecode Java ou adulterando recursos. Este processo é conhecido como _modding_ na cena de hacking de jogos móveis. Os patches podem ser aplicados de várias maneiras, incluindo edição de arquivos binários em um editor hexadecimal e descompilação, edição e re-montagem de um aplicativo. Daremos exemplos detalhados de patches úteis em capítulos posteriores.

Tenha em mente que os sistemas operacionais móveis modernos impõem estritamente a assinatura de código, portanto, executar aplicativos modificados não é tão direto quanto costumava ser em ambientes desktop. Os especialistas em segurança tinham uma vida muito mais fácil nos anos 90! Felizmente, a aplicação de patches não é muito difícil se você trabalhar em seu próprio dispositivo. Você simplesmente tem que re-assinar o aplicativo ou desabilitar as facilidades padrão de verificação de assinatura de código para executar código modificado.

### Injeção de Código

A injeção de código é uma técnica muito poderosa que permite explorar e modificar processos em tempo de execução. A injeção pode ser implementada de várias maneiras, mas você se sairá bem sem conhecer todos os detalhes, graças a ferramentas gratuitas, bem documentadas e disponíveis que automatizam o processo. Essas ferramentas dão a você acesso direto à memória do processo e estruturas importantes, como objetos em tempo real instanciados pelo aplicativo. Elas vêm com muitas funções utilitárias que são úteis para resolver bibliotecas carregadas, conectar métodos e funções nativas e muito mais. A adulteração da memória do processo é mais difícil de detectar do que a aplicação de patches em arquivos, por isso é o método preferido na maioria dos casos.

@MASTG-TOOL-0139, @MASTG-TOOL-0031 e @MASTG-TOOL-0027 são as estruturas de injeção de código e conexão mais amplamente usadas na indústria móvel. As três estruturas diferem em filosofia de design e detalhes de implementação: ElleKit e Xposed focam em injeção de código e/ou conexão, enquanto o Frida visa ser uma "estrutura de instrumentação dinâmica" completa, incorporando injeção de código, vinculações de linguagem e uma VM e console JavaScript injetáveis.

Incluiremos exemplos de todas as três estruturas. Recomendamos começar com o Frida porque é o mais versátil dos três (por esta razão, também incluiremos mais detalhes e exemplos do Frida). Notavelmente, o Frida pode injetar uma VM JavaScript em um processo tanto no Android quanto no iOS, enquanto a injeção com ElleKit só funciona no iOS e o Xposed só funciona no Android. No final, no entanto, você pode, é claro, alcançar muitos dos mesmos objetivos com qualquer estrutura.

## Análise Binária Estática e Dinâmica

A engenharia reversa é o processo de reconstruir a semântica do código-fonte de um programa compilado. Em outras palavras, você desmonta o programa, executa-o, simula partes dele e faz outras coisas indizíveis para entender o que ele faz e como.

### Usando Desmontadores e Descompiladores

Desmontadores e descompiladores permitem que você traduza o código binário ou bytecode de um aplicativo de volta para um formato mais ou menos compreensível. Ao usar essas ferramentas em binários nativos, você pode obter código assembly que corresponde à arquitetura para a qual o aplicativo foi compilado. Desmontadores convertem código de máquina para código assembly que por sua vez é usado por descompiladores para gerar código equivalente de linguagem de alto nível. Aplicativos Java Android podem ser desmontados para smali, que é uma linguagem assembly para o formato DEX usado pelo Dalvik, a VM Java do Android. O assembly Smali também pode ser facilmente descompilado de volta para código Java equivalente.

Em teoria, o mapeamento entre assembly e código de máquina deve ser um-para-um e, portanto, pode dar a impressão de que desmontar é uma tarefa simples. Mas na prática, existem múltiplas armadilhas, como:

- Distinção confiável entre código e dados.
- Tamanho variável de instrução.
- Instruções de branch indiretas.
- Funções sem instruções CALL explícitas dentro do segmento de código do executável.
- Sequências de código independente de posição (PIC).
- Código assembly feito à mão.

Da mesma forma, a descompilação é um processo muito complicado, envolvendo muitas abordagens determinísticas e baseadas em heurísticas. Como consequência, a descompilação geralmente não é muito precisa, mas mesmo assim é muito útil para obter uma compreensão rápida da função que está sendo analisada. A precisão da descompilação depende da quantidade de informação disponível no código que está sendo descompilado e da sofisticação do descompilador. Além disso, muitas ferramentas de compilação e pós-compilação introduzem complexidade adicional ao código compilado para aumentar a dificuldade de compreensão e/ou até mesmo da própria descompilação. Tal código é referido como [_código ofuscado_](#obfuscation).

Ao longo das últimas décadas, muitas ferramentas aperfeiçoaram o processo de desmontagem e descompilação, produzindo saída com alta fidelidade. Instruções de uso avançado para qualquer uma das ferramentas disponíveis podem facilmente preencher um livro próprio. A melhor maneira de começar é simplesmente pegar uma ferramenta que se adapte às suas necessidades e orçamento e obter um guia do usuário bem avaliado. Nesta seção, forneceremos uma introdução a algumas dessas ferramentas e nos capítulos subsequentes de "Engenharia Reversa e Adulteração" do Android e iOS, focaremos nas técnicas em si, especialmente aquelas que são específicas da plataforma em questão.

### Ofuscação

A ofuscação é o processo de transformar código e dados para torná-los mais difíceis de compreender (e às vezes até difíceis de desmontar). Geralmente é uma parte integral do esquema de proteção de software. A ofuscação não é algo que pode ser simplesmente ligado ou desligado, programas podem ser tornados incompreensíveis, no todo ou em parte, de muitas maneiras e em diferentes graus.

> Nota: Todas as técnicas apresentadas abaixo não impedirão alguém com tempo e orçamento suficientes de fazer engenharia reversa do seu aplicativo. No entanto, combinar essas técnicas tornará seu trabalho significativamente mais difícil. O objetivo é, portanto, desencorajar engenheiros reversos de realizar análises adicionais e não valer a pena o esforço.

As seguintes técnicas podem ser usadas para ofuscar um aplicativo:

- Ofuscação de nomes
- Substituição de instruções
- Achatamento de fluxo de controle
- Injeção de código morto
- Criptografia de strings
- Empacotamento

#### Ofuscação de Nomes

O compilador padrão gera símbolos binários com base em nomes de classes e funções do código-fonte. Portanto, se nenhuma ofuscação for aplicada, os nomes dos símbolos permanecem significativos e podem ser facilmente extraídos do binário do aplicativo. Por exemplo, uma função que detecta jailbreak pode ser localizada pesquisando por palavras-chave relevantes (por exemplo, "jailbreak"). A listagem abaixo mostra a função desmontada `JailbreakDetectionViewController.jailbreakTest4Tapped` do @MASTG-APP-0024.

```assembly
__T07DVIA_v232JailbreakDetectionViewControllerC20jailbreakTest4TappedyypF:
stp        x22, x21, [sp, #-0x30]!
mov        rbp, rsp
```

Após a ofuscação, podemos observar que o nome do símbolo não é mais significativo, como mostrado na listagem abaixo.

```assembly
__T07DVIA_v232zNNtWKQptikYUBNBgfFVMjSkvRdhhnbyyFySbyypF:
stp        x22, x21, [sp, #-0x30]!
mov        rbp, rsp
```

No entanto, isso só se aplica aos nomes de funções, classes e campos. O código real permanece inalterado, então um atacante ainda pode ler a versão desmontada da função e tentar entender seu propósito (por exemplo, para recuperar a lógica de um algoritmo de segurança).

#### Substituição de Instruções

Esta técnica substitui operadores binários padrão como adição ou subtração por representações mais complexas. Por exemplo, uma adição `x = a + b` pode ser representada como `x = -(-a) - (-b)`. No entanto, usar a mesma representação de substituição poderia ser facilmente revertido, então é recomendado adicionar múltiplas técnicas de substituição para um único caso e introduzir um fator aleatório
. No entanto, usar a mesma representação de substituição poderia ser facilmente revertido, então é recomendado adicionar múltiplas técnicas de substituição para um único caso e introduzir um fator aleatório. Esta técnica pode ser revertida durante a descompilação, mas dependendo da complexidade e profundidade das substituições, revertê-la ainda pode ser demorado.

#### Achatamento de Fluxo de Controle

O achatamento de fluxo de controle substitui o código original por uma representação mais complexa. A transformação quebra o corpo de uma função em blocos básicos e os coloca todos dentro de um único loop infinito com uma instrução switch que controla o fluxo do programa. Isso torna o fluxo do programa significativamente mais difícil de seguir porque remove os construtos condicionais naturais que geralmente tornam o código mais fácil de ler.

<img src="Images/Chapters/0x06j/control-flow-flattening.png" width="100%" />

A imagem mostra como o achatamento de fluxo de controle altera o código. Veja ["Obfuscating C++ programs via control flow flattening"](https://web.archive.org/web/20240414202600/http://ac.inf.elte.hu/Vol_030_2009/003.pdf) para mais informações.

#### Injeção de Código Morto

Esta técnica torna o fluxo de controle do programa mais complexo injetando código morto no programa. Código morto é um trecho de código que não afeta o comportamento do programa original, mas aumenta a sobrecarga do processo de engenharia reversa.

#### Criptografia de Strings

Aplicativos são frequentemente compilados com chaves embutidas, licenças, tokens e URLs de endpoint. Por padrão, todos eles são armazenados em texto simples na seção de dados do binário de um aplicativo. Esta técnica criptografa esses valores e injeta trechos de código no programa que descriptografarão esses dados antes de serem usados pelo programa.

#### Empacotamento

[Empacotamento](https://attack.mitre.org/techniques/T1027/002/) é uma técnica de ofuscação de reescrita dinâmica que comprime ou criptografa o executável original em dados e o recupera dinamicamente durante a execução. Empacotar um executável altera a assinatura do arquivo na tentativa de evitar detecção baseada em assinatura.

### Depuração e Rastreamento

No sentido tradicional, a depuração é o processo de identificar e isolar problemas em um programa como parte do ciclo de vida do desenvolvimento de software. As mesmas ferramentas usadas para depuração são valiosas para engenheiros reversos, mesmo quando identificar bugs não é o objetivo principal. Depuradores permitem a suspensão do programa em qualquer ponto durante o tempo de execução, inspeção do estado interno do processo e até mesmo modificação de registradores e memória. Essas habilidades simplificam a inspeção do programa.

_Depuração_ geralmente significa sessões interativas de depuração nas quais um depurador é anexado ao processo em execução. Em contraste, _rastreamento_ refere-se ao registro passivo de informações sobre a execução do aplicativo (como chamadas de API). O rastreamento pode ser feito de várias maneiras, incluindo APIs de depuração, hooks de função e facilidades de rastreamento do Kernel. Novamente, cobriremos muitas dessas técnicas nos capítulos específicos do sistema operacional "Engenharia Reversa e Adulteração".

## Técnicas Avançadas

Para tarefas mais complicadas, como desofuscar binários fortemente ofuscados, você não irá longe sem automatizar certas partes da análise. Por exemplo, entender e simplificar um grafo de fluxo de controle complexo com base em análise manual no desmontador levaria anos (e muito provavelmente o deixaria louco muito antes de terminar). Em vez disso, você pode aumentar seu fluxo de trabalho com ferramentas personalizadas. Felizmente, desmontadores modernos vêm com APIs de script e extensão, e muitas extensões úteis estão disponíveis para desmontadores populares. Também existem motores de desmontagem de código aberto e estruturas de análise binária.

Como sempre no hacking, a regra do vale-tudo se aplica: simplesmente use o que for mais eficiente. Cada binário é diferente, e todos os engenheiros reversos têm seu próprio estilo. Muitas vezes, a melhor maneira de alcançar seu objetivo é combinar abordagens (como rastreamento baseado em emulador e execução simbólica). Para começar, escolha um bom desmontador e/ou estrutura de engenharia reversa, então familiarize-se com seus recursos particulares e APIs de extensão. No final, a melhor maneira de melhorar é obter experiência prática.

### Instrumentação Binária Dinâmica

Outra abordagem útil para binários nativos é a instrumentação binária dinâmica (DBI). Estruturas de instrumentação como Valgrind e PIN suportam rastreamento de nível de instrução de granulação fina de processos únicos. Isso é realizado inserindo código gerado dinamicamente em tempo de execução. O Valgrind compila bem no Android, e binários pré-compilados estão disponíveis para download.

O [README do Valgrind](http://valgrind.org/docs/manual/dist.readme-android.html "Valgrind README") inclui instruções específicas de compilação para Android.

### Análise Dinâmica Baseada em Emulação

Emulação é uma imitação de uma certa plataforma de computador ou programa sendo executado em uma plataforma diferente ou dentro de outro programa. O software ou hardware que realiza esta imitação é chamado de _emulador_. Emuladores fornecem uma alternativa muito mais barata a um dispositivo real, onde um usuário pode manipulá-lo sem se preocupar em danificar o dispositivo. Existem múltiplos emuladores disponíveis para Android, mas para iOS praticamente não existem emuladores viáveis disponíveis. O iOS só tem um simulador, enviado dentro do Xcode.

A diferença entre um simulador e um emulador frequentemente causa confusão e leva ao uso dos dois termos de forma intercambiável, mas na realidade eles são diferentes, especialmente para o caso de uso do iOS. Um emulador imita tanto o ambiente de software quanto o de hardware de uma plataforma alvo. Por outro lado, um simulador só imita o ambiente de software.

Emuladores baseados em QEMU para Android levam em consideração a RAM, CPU, desempenho da bateria etc (componentes de hardware) ao executar um aplicativo, mas em um simulador iOS este comportamento de componente de hardware não é levado em consideração. O simulador iOS até carece da implementação do kernel do iOS, como resultado, se um aplicativo estiver usando syscalls, ele não pode ser executado neste simulador.

Em palavras simples, um emulador é uma imitação muito mais próxima da plataforma alvo, enquanto um simulador imita apenas uma parte dela.

Executar um aplicativo no emulador dá a você maneiras poderosas de monitorar e manipular seu ambiente. Para algumas tarefas de engenharia reversa, especialmente aquelas que requerem rastreamento de instruções de baixo nível, a emulação é a melhor (ou única) escolha. Infelizmente, este tipo de análise só é viável para Android, porque nenhum emulador gratuito ou de código aberto existe para iOS (o simulador iOS não é um emulador, e aplicativos compilados para um dispositivo iOS não são executados nele). O único emulador iOS disponível é uma solução comercial SaaS - @MASTG-TOOL-0108.

### Ferramentas Personalizadas com Estruturas de Engenharia Reversa

Embora a maioria dos desmontadores baseados em GUI profissionais apresentem facilidades de script e extensibilidade, eles simplesmente não são bem adequados para resolver problemas particulares. Estruturas de engenharia reversa permitem que você realize e automatize qualquer tipo de tarefa de reversão sem depender de uma GUI pesada. Notavelmente, a maioria das estruturas de reversão são de código aberto e/ou disponíveis gratuitamente. Estruturas populares com suporte para arquiteturas móveis incluem @MASTG-TOOL-0073 e @MASTG-TOOL-0030.

#### Exemplo: Análise de Programa com Execução Simbólica/Concolica

No final dos anos 2000, testes baseados em execução simbólica tornaram-se uma maneira popular de identificar vulnerabilidades de segurança. Execução "simbólica" na verdade se refere ao processo de representar caminhos possíveis através de um programa como fórmulas em lógica de primeira ordem. Solucionadores de Satisfiability Modulo Theories (SMT) são usados para verificar a satisfatibilidade dessas fórmulas e fornecer soluções, incluindo valores concretos das variáveis necessárias para alcançar um certo ponto de execução no caminho correspondente à fórmula resolvida.

Em palavras simples, a execução simbólica é analisar matematicamente um programa sem executá-lo. Durante a análise, cada entrada desconhecida é representada como uma variável matemática (um valor simbólico), e portanto todas as operações realizadas nessas variáveis são registradas como uma árvore de operações (também conhecida como AST (árvore sintática abstrata), da teoria de compiladores). Essas ASTs podem ser traduzidas para as chamadas _restrições_ que serão interpretadas por um solucionador SMT. No final desta análise, uma equação matemática final é obtida, na qual as variáveis são as entradas cujos valores não são conhecidos. Solucionadores SMT são programas especiais que resolvem essas equações para dar valores possíveis para as variáveis de entrada dado um estado final.

Para ilustrar isso, imagine uma função que recebe uma entrada (`x`) e a multiplica pelo valor de uma segunda entrada (`y`). Finalmente, há uma condição _if_ que verifica se o valor calculado é maior que o valor de uma variável externa(`z`), e retorna "sucesso" se verdadeiro, caso contrário retorna "falha". A equação para esta operação será `(x * y) > z`.

Se quisermos que a função sempre retorne "sucesso" (estado final), podemos dizer ao solucionador SMT para calcular os valores para `x` e `y` (variáveis de entrada) que satisfazem a equação correspondente. Como é o caso de variáveis globais, seu valor pode ser alterado de fora desta função, o que pode levar a saídas diferentes sempre que esta função for executada. Isso adiciona complexidade adicional na determinação da solução correta.

Internamente, solucionadores SMT usam várias técnicas de resolução de equações para gerar solução para tais equações. Algumas das técnicas são muito avançadas e sua discussão está além do escopo deste livro.

Em uma situação do mundo real, as funções são muito mais complexas do que o exemplo acima. A complexidade aumentada das funções pode representar desafios significativos para a execução simbólica clássica. Alguns dos desafios são resumidos abaixo:

- Loops e recursões em um programa podem levar a _árvore de execução infinita_.
- Múltiplos branches condicionais ou condições aninhadas podem levar a _explosão de caminhos_.
- Equações complexas geradas por execução simbólica podem não ser solucionáveis por solucionadores SMT devido às suas limitações.
- O programa está usando chamadas de sistema, chamadas de biblioteca ou eventos de rede que não podem ser tratados por execução simbólica.

Para superar esses desafios, tipicamente, a execução simbólica é combinada com outras técnicas, como _execução dinâmica_ (também chamada de _execução concreta_) para mitigar o problema de explosão de caminhos específico da execução simbólica clássica. Esta combinação de execução concreta (real) e simbólica é referida como _execução concolica_ (o nome concolic vem de **conc**reto e simb**ólico**), às vezes também chamada de _execução simbólica dinâmica_.

Para visualizar isso, no exemplo acima, podemos obter o valor da variável externa realizando mais engenharia reversa ou executando dinamicamente o programa e alimentando esta informação em nossa análise de execução simbólica. Esta informação extra reduzirá a complexidade de nossas equações e pode produzir resultados de análise mais precisos. Juntamente com solucionadores SMT melhorados e velocidades de hardware atuais, a execução concolica permite explorar caminhos em módulos de software de tamanho médio (ou seja, na ordem de 10 KLOC).

Além disso, a execução simbólica também é útil para apoiar tarefas de desofuscação, como simplificar grafos de fluxo de controle. Por exemplo, Jonathan Salwan e Romain Thomas [mostraram como fazer engenharia reversa de proteções de software baseadas em VM usando Execução Simbólica Dinâmica](https://drive.google.com/file/d/1EzuddBA61jEMy8XbjQKFF3jyoKwW7tLq/view?usp=sharing "Jonathan Salwan and Romain Thomas: How Triton can help to reverse virtual machine based software protections") [#salwan] (ou seja, usando uma mistura de traços de execução reais, simulação e execução simbólica).

Na seção Android, você encontrará um passo a passo para quebrar uma verificação de licença simples em um aplicativo Android usando execução simbólica.

## Referências

- [#vadla] Ole André Vadla Ravnås, Anatomy of a code tracer - <https://medium.com/@oleavr/anatomy-of-a-code-tracer-b081aadb0df8>
- [#salwan] Jonathan Salwan and Romain Thomas, How Triton can help to reverse virtual machine based software protections - <https://drive.google.com/file/d/1EzuddBA61jEMy8XbjQKFF3jyoKwW7tLq/view?usp=sharing>