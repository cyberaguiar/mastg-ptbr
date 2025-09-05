
# Teste de Segurança de Aplicativos Móveis

Nas seções a seguir, forneceremos uma breve visão geral dos princípios gerais de teste de segurança e terminologia chave. Os conceitos introduzidos são amplamente idênticos aos encontrados em outros tipos de teste de penetração, então se você é um testador experiente pode estar familiarizado com algum do conteúdo.

Ao longo do guia, usamos "teste de segurança de aplicativos móveis" como uma frase abrangente para se referir à avaliação da segurança de aplicativos móveis via análise estática e dinâmica. Termos como "teste de penetração de aplicativos móveis" e "revisão de segurança de aplicativos móveis" são usados de forma um tanto inconsistente na indústria de segurança, mas esses termos se referem aproximadamente à mesma coisa. Um teste de segurança de aplicativo móvel geralmente é parte de uma avaliação de segurança maior ou teste de penetração que abrange a arquitetura cliente-servidor e APIs do lado do servidor usadas pelo aplicativo móvel.

Neste guia, cobrimos teste de segurança de aplicativos móveis em dois contextos. O primeiro é o teste de segurança "clássico" concluído próximo ao final do ciclo de vida de desenvolvimento. Neste contexto, o testador acessa uma versão quase finalizada ou pronta para produção do aplicativo, identifica problemas de segurança e escreve um relatório (geralmente devastador). O outro contexto é caracterizado pela implementação de requisitos e automação de testes de segurança desde o início do ciclo de vida de desenvolvimento de software em diante. Os mesmos requisitos básicos e casos de teste se aplicam a ambos os contextos, mas o método de alto nível e o nível de interação com o cliente diferem.

## Princípios de Teste

### Teste White-box versus Black-box

Vamos começar definindo os conceitos:

- **Teste Black-box** é conduzido sem que o testador tenha qualquer informação sobre o aplicativo sendo testado. Este processo às vezes é chamado de "teste de conhecimento zero". O principal propósito deste teste é permitir que o testador se comporte como um atacante real no sentido de explorar usos possíveis para informações publicamente disponíveis e descobríveis.
- **Teste White-box** (às vezes chamado de "teste de conhecimento completo") é o total oposto do teste black-box no sentido de que o testador tem conhecimento completo do aplicativo. O conhecimento pode abranger código-fonte, documentação e diagramas. Esta abordagem permite testes muito mais rápidos que o teste black-box devido à sua transparência e com o conhecimento adicional ganho, um testador pode construir casos de teste muito mais sofisticados e granulares.
- **Teste Gray-box** é todo teste que cai entre os dois tipos de teste mencionados anteriormente: alguma informação é fornecida ao testador (geralmente apenas credenciais), e outras informações devem ser descobertas. Este tipo de teste é um compromisso interessante no número de casos de teste, custo, velocidade e escopo do teste. Teste gray-box é o tipo mais comum de teste na indústria de segurança.

Recomendamos fortemente que você solicite o código-fonte para que possa usar o tempo de teste da forma mais eficiente possível. O acesso ao código pelo testador obviamente não simula um ataque externo, mas simplifica a identificação de vulnerabilidades permitindo que o testador verifique cada anomalia identificada ou comportamento suspeito no nível do código. Um teste white-box é o caminho a seguir se o aplicativo não foi testado antes.

Mesmo que descompilar no Android seja direto, o código-fonte pode ser ofuscado, e desofuscar será demorado. Restrições de tempo são, portanto, outra razão para o testador ter acesso ao código-fonte.

### Análise de Vulnerabilidade

Análise de vulnerabilidade é geralmente o processo de procurar vulnerabilidades em um aplicativo. Embora isso possa ser feito manualmente, scanners automatizados são geralmente usados para identificar as principais vulnerabilidades. Análise estática e dinâmica são tipos de análise de vulnerabilidade.

### Análise Estática versus Dinâmica

Static Application Security Testing (SAST) envolve examinar os componentes de um aplicativo sem executá-los, analisando o código-fonte manualmente ou automaticamente.
A OWASP fornece informações sobre [Análise de Código Estático](https://owasp.org/www-community/controls/Static_Code_Analysis "OWASP Static Code Analysis") que podem ajudá-lo a entender técnicas, pontos fortes, fraquezas e limitações.

Dynamic Application Security Testing (DAST) envolve examinar o aplicativo durante o tempo de execução. Este tipo de análise pode ser manual ou automático. Geralmente não fornece a informação que a análise estática fornece, mas é uma boa maneira de detectar elementos interessantes (ativos, recursos, pontos de entrada, etc.) do ponto de vista do usuário.

Agora que definimos análise estática e dinâmica, vamos mergulhar mais fundo.

### Análise Estática

Durante a análise estática, o código-fonte do aplicativo móvel é revisado para garantir a implementação apropriada de controles de segurança. Na maioria dos casos, uma abordagem híbrida automática/manual é usada. Scans automáticos pegam as frutas mais baixas, e o testador humano pode explorar a base de código com contextos de uso específicos em mente.

#### Revisão Manual de Código

Um testador realiza revisão manual de código analisando manualmente o código-fonte do aplicativo móvel em busca de vulnerabilidades de segurança. Os métodos variam de uma busca básica por palavras-chave via comando 'grep' a um exame linha por linha do código-fonte. IDEs (Ambientes de Desenvolvimento Integrados) frequentemente fornecem funções básicas de revisão de código e podem ser estendidos com várias ferramentas.

Uma abordagem comum à análise manual de código envolve identificar indicadores-chave de vulnerabilidade de segurança pesquisando por certas APIs e palavras-chave, como chamadas de método relacionadas a banco de dados como "executeStatement" ou "executeQuery". Código contendo essas strings é um bom ponto de partida para análise manual.

Em contraste com a análise automática de código, a revisão manual de código é muito boa para identificar vulnerabilidades na lógica de negócios, violações de padrões e falhas de design, especialmente quando o código é tecnicamente seguro mas logicamente falho. Tais cenários são improváveis de serem detectados por qualquer ferramenta de análise automática de código.

Uma revisão manual de código requer um revisor de código especialista que seja proficiente tanto na linguagem quanto nos frameworks usados para o aplicativo móvel. Revisão completa de código pode ser um processo lento, tedioso e demorado para o revisor, especialmente considerando grandes bases de código com muitas dependências.

#### Análise Automatizada de Código-Fonte

Ferramentas de análise automatizadas podem ser usadas para acelerar o processo de revisão do Static Application Security Testing (SAST). Elas verificam o código-fonte para conformidade com um conjunto predefinido de regras ou melhores práticas da indústria, então tipicamente exibem uma lista de achados ou avisos e flags para todas as violações detectadas. Algumas ferramentas de análise estática executam apenas contra o aplicativo compilado, algumas devem ser alimentadas com o código-fonte original, e algumas executam como plugins de análise ao vivo no Integrated Development Environment (IDE).

Embora algumas ferramentas de análise estática de código incorporem muita informação sobre as regras e semântica necessárias para analisar aplicativos móveis, elas podem produzir muitos falsos positivos, particularmente se não estiverem configuradas para o ambiente alvo. Um profissional de segurança deve, portanto, sempre revisar os resultados.

### Análise Dinâmica

O foco do DAST é o teste e avaliação de aplicativos via sua execução em tempo real. O principal objetivo da análise dinâmica é encontrar vulnerabilidades de segurança ou pontos fracos em um programa enquanto ele está executando. A análise dinâmica é conduzida tanto na camada da plataforma móvel quanto contra os serviços backend e APIs, onde os padrões de requisição e resposta do aplicativo móvel podem ser analisados.

A análise dinâmica é geralmente usada para verificar mecanismos de segurança que fornecem proteção suficiente contra os tipos mais prevalentes de ataque, como divulgação de dados em trânsito, problemas de autenticação e autorização, e erros de configuração de servidor.

### Evitando Falsos Positivos

#### Ferramentas de Scanning Automatizado

A falta de sensibilidade das ferramentas de teste automatizado ao contexto do aplicativo é um desafio. Essas ferramentas podem identificar um problema potencial que é irrelevante. Tais resultados são chamados de "falsos positivos".

Por exemplo, testadores de segurança comumente relatam vulnerabilidades que são exploráveis em um navegador web mas não são relevantes para o aplicativo móvel. Este falso positivo ocorre porque ferramentas automatizadas usadas para escanear o serviço backend são baseadas em aplicativos web regulares baseados em navegador. Problemas como CSRF (Cross-site Request Forgery) e Cross-Site Scripting (XSS) são relatados de acordo.

Vamos tomar CSRF como exemplo. Um ataque CSRF bem-sucedido requer o seguinte:

- A capacidade de atrair o usuário logado a abrir um link malicioso no navegador web usado para acessar o site vulnerável.
- O cliente (navegador) deve adicionar automaticamente o cookie de sessão ou outro token de autenticação à requisição.

Aplicativos móveis não cumprem esses requisitos: mesmo se WebViews e gerenciamento de sessão baseado em cookie forem usados, qualquer link malicioso que o usuário clicar abre no navegador padrão, que tem um armazenamento de cookie separado.

Cross-Site Scripting Armazenado (XSS) pode ser um problema se o aplicativo incluir WebViews, e pode até levar à execução de comandos se o aplicativo exportar interfaces JavaScript. No entanto, Cross-Site Scripting Refletido raramente é um problema pela razão mencionada acima (embora seja discutível se eles deveriam existir, escapar a saída é simplesmente uma melhor prática).

> Em qualquer caso, considere cenários de exploração quando realizar a avaliação de risco; não confie cegamente na saída da sua ferramenta de scanning.

### Teste de Penetração (a.k.a. Pentesting)

A abordagem clássica envolve teste de segurança completo da build final ou quase final do aplicativo, ex., a build que está disponível no final do processo de desenvolvimento. Para teste no final do processo de desenvolvimento, recomendamos o [Mobile App Security Verification Standard (MASVS)](https://github.com/OWASP/masvs "OWASP MASVS") e a lista de verificação associada como base para teste. Um teste de segurança típico é estruturado da seguinte forma:

- **Preparação** - definindo o escopo do teste de segurança, incluindo identificação de controles de segurança aplicáveis, objetivos de teste da organização e dados sensíveis. Mais geralmente, preparação inclui toda sincronização com o cliente, bem como proteger legalmente o testador (que frequentemente é uma terceira parte). Lembre-se, atacar um sistema sem autorização por escrito é ilegal em muitas partes do mundo!
- **Coleta de Inteligência** - analisando o contexto **ambiental** e **arquitetural** do aplicativo para obter um entendimento contextual geral.
- **Mapeamento do Aplicativo** - baseado em informação das fases anteriores; pode ser complementado por scanning automatizado e explorando manualmente o aplicativo. Mapeamento fornece um entendimento completo do aplicativo, seus pontos de entrada, os dados que ele mantém e as principais vulnerabilidades potenciais. Essas vulnerabilidades podem então ser classificadas de acordo com o dano que sua exploração causaria para que o testador de segurança possa priorizá-las. Esta fase inclui a criação de casos de teste que podem ser usados durante a execução do teste.
- **Exploração** - nesta fase, o testador de segurança tenta penetrar o aplicativo explorando as vulnerabilidades identificadas durante a fase anterior. Esta fase é necessária para determinar se as vulnerabilidades são reais e verdadeiros positivos.
- **Relatório** - nesta fase, que é essencial para o cliente, o testador de segurança reporta as vulnerabilidades. Isso inclui o processo de exploração em detalhe, classifica o tipo de vulnerabilidade, documenta o risco se um atacante fosse capaz de comprometer o alvo e descreve quais dados o testador foi capaz de acessar ilegitimamente.

#### Preparação

O nível de segurança no qual o aplicativo será testado deve ser decidido antes do teste. Os requisitos de segurança devem ser decididos no início do projeto. Diferentes organizações têm diferentes necessidades de segurança e recursos disponíveis para investir em atividades de teste. Embora os testes no perfil MAS-L1 sejam aplicáveis a todos os aplicativos móveis, percorrer todos os testes MAS-L1 e MAS-L2 com partes interessadas técnicas e de negócios é uma boa maneira de decidir sobre um nível de cobertura de teste.

Organizações podem ter diferentes obrigações regulatórias e legais em certos territórios. Mesmo se um aplicativo não lidar com dados sensíveis, alguns testes MAS-L2 podem ser relevantes (devido a regulamentos da indústria ou leis locais). Por exemplo, autenticação de dois fatores (2FA) pode ser obrigatória para um aplicativo financeiro e imposta pelo banco central de um país e/ou autoridades reguladoras financeiras.

Objetivos/controles de segurança definidos anteriormente no processo de desenvolvimento também podem ser revisados durante a discussão com as partes interessadas. Alguns controles podem estar em conformidade com perfis MAS, mas outros podem ser específicos para a organização ou aplicativo.

Todas as partes envolvidas devem concordar com as decisões e o escopo na lista de verificação porque estes definirão a base para todos os testes de segurança.

##### Coordenação com o Cliente

Configurar um ambiente de teste funcional pode ser uma tarefa desafiadora. Por exemplo, restrições nos pontos de acesso sem fio empresariais e redes podem impedir a análise dinâmica realizada nas instalações do cliente. Políticas da empresa podem proibir o uso de telefones com root ou ferramentas de teste de rede (hardware e software) dentro de redes empresariais. Aplicativos que implementam detecção de root e outras contramedidas de engenharia reversa podem aumentar significativamente o trabalho necessário para análise adicional.

Teste de segurança envolve muitas tarefas invasivas, incluindo monitoramento e manipulação do tráfego de rede do aplicativo móvel, inspeção dos arquivos de dados do aplicativo e instrumentação de chamadas de API. Controles de segurança, como certificate pinning e detecção de root, podem impedir essas tarefas e dramaticamente desacelerar o teste.

Para superar esses obstáculos, você pode querer solicitar duas variantes de build do aplicativo da equipe de desenvolvimento. Uma variante deve ser uma build de release para que você possa determinar se os controles implementados estão funcionando adequadamente e não podem ser contornados facilmente. A segunda variante deve ser uma build de debug para a qual certos controles de segurança foram desativados. Testar duas builds diferentes é a maneira mais eficiente de cobrir todos os casos de teste.

Dependendo do escopo do engajamento, esta abordagem pode não ser possível. Solicitar ambas as builds de produção e debug para um teste white-box ajudará você a completar todos os casos de teste e declarar claramente a maturidade de segurança do aplicativo. O cliente pode preferir que testes black-box sejam focados no aplicativo de produção e na avaliação da efetividade de seus controles de segurança.

O escopo de ambos os tipos de teste deve ser discutido durante a fase de preparação. Por exemplo, se os controles de segurança devem ser ajustados deve ser decidido antes do teste. Tópicos adicionais são discutidos abaixo.

##### Identificando Dados Sensíveis

Classificações de informação sensível diferem por indústria e país. Além disso, organizações podem ter uma visão restritiva de dados sensíveis, e elas podem ter uma política de classificação de dados que define claramente informação sensível.

Há três estados gerais dos quais os dados podem ser acessíveis:

- **Em repouso** - os dados estão sentados em um arquivo ou armazenamento de dados
- **Em uso** - um aplicativo carregou os dados em seu espaço de endereço
- **Em trânsito** - dados foram trocados entre aplicativo móvel e endpoint ou processos consumidores no dispositivo, ex., durante IPC (Comunicação Entre Processos)

O grau de escrutínio que é apropriado para cada estado pode depender da importância dos dados e probabilidade de serem acessados. Por exemplo, dados mantidos na memória do aplicativo podem ser mais vulneráveis que dados em servidores web ao acesso via core dumps porque atacantes são mais propensos a obter acesso físico a dispositivos móveis que a servidores web.

Quando nenhuma política de classificação de dados está disponível, use a seguinte lista de informação que é geralmente considerada sensível:

- informação de autenticação do usu

(credenciais, PINs, etc.)
- Informação Pessoalmente Identificável (PII) que pode ser abusada para roubo de identidade: números de seguro social, números de cartão de crédito, números de conta bancária, informação de saúde
- identificadores de dispositivo que podem identificar uma pessoa
- dados altamente sensíveis cujo comprometimento levaria a danos reputacionais e/ou custos financeiros
- quaisquer dados cuja proteção é uma obrigação legal
- quaisquer dados técnicos gerados pelo aplicativo (ou seus sistemas relacionados) e usados para proteger outros dados ou o próprio sistema (ex., chaves de criptografia)

Uma definição de "dados sensíveis" deve ser decidida antes do teste começar porque detectar vazamento de dados sensíveis sem uma definição pode ser impossível.

##### Identificando Contextos Relevantes para Segurança no Código

Ao desenvolver um aplicativo móvel, é crucial identificar e lidar com precisão com contextos relevantes para segurança dentro da base de código. Esses contextos tipicamente envolvem operações como autenticação, criptografia e autorização, que são frequentemente o alvo de ataques de segurança. Implementação incorreta de funções criptográficas nessas áreas pode levar a vulnerabilidades de segurança significativas.

Distinguir adequadamente contextos relevantes para segurança ajuda a minimizar falsos positivos durante testes de segurança. Falsos positivos podem desviar a atenção de problemas reais e desperdiçar recursos valiosos. Aqui estão alguns cenários comuns:

- **Geração de Números Aleatórios**: Usar geradores de números aleatórios previsíveis pode ser uma falha de segurança séria em contextos como autenticação ou geração de chaves de criptografia. No entanto, nem todos os usos de números aleatórios são sensíveis à segurança. Por exemplo, usar um gerador de números aleatórios menos robusto para propósitos não relacionados à segurança como embaralhar uma lista de itens em um jogo é geralmente aceitável.

- **Hashing**: Hashing é frequentemente usado em segurança para armazenar senhas ou garantir integridade de dados. No entanto, fazer hash de um valor não sensível, como a resolução de tela de um dispositivo para analytics, não é uma preocupação de segurança.

- **Criptografia vs Codificação**: Um mal-entendido comum é confundir codificação (como Base64) com criptografia. Codificação Base64 não é um método seguro para proteger dados sensíveis pois é facilmente reversível. É crucial reconhecer quando dados requerem criptografia real (para confidencialidade) versus quando estão sendo codificados para compatibilidade ou formatação (como codificar dados binários em formato de texto para transmissão). Interpretar erroneamente codificação como uma medida de segurança pode levar a negligenciar necessidades reais de criptografia para dados sensíveis.

- **Armazenamento de Token de API**: Armazenar tokens de API ou chaves em texto simples dentro do código do aplicativo ou em locais inseguros (como SharedPreferences no Android ou UserDefaults no iOS) é um erro de segurança comum. No entanto, se o token é para uma API pública não sensível e somente leitura, isso pode não ser um risco de segurança. Contrast

isso com armazenar um token para uma API sensível ou com acesso de escrita, onde armazenamento impróprio seria uma preocupação de segurança significativa.

#### Coleta de Inteligência

Coleta de inteligência envolve a coleta de informação sobre a arquitetura do aplicativo, os casos de uso de negócios que o aplicativo serve e o contexto no qual o aplicativo opera. Tal informação pode ser classificada como "ambiental" ou "arquitetural".

##### Informação Ambiental

Informação ambiental inclui:

- Os objetivos da organização para o aplicativo. Funcionalidade molda a interação dos usuários com o aplicativo e pode tornar algumas superfícies mais prováveis que outras de serem alvo de atacantes.
- A indústria relevante. Diferentes indústrias podem ter diferentes perfis de risco.
- Partes interessadas e investidores; entendendo quem está interessado e responsável pelo aplicativo.
- Processos internos, fluxos de trabalho e estruturas organizacionais. Processos e fluxos de trabalho internos específicos da organização podem criar oportunidades para [vulnerabilidades de lógica de negócios](https://owasp.org/www-community/vulnerabilities/Business_logic_vulnerability "Business logic vulnerability").

##### Informação Arquitetural

Informação arquitetural inclui:

- **O aplicativo móvel:** Como o aplicativo acessa dados e os gerencia em processo, como se comunica com outros recursos e gerencia sessões de usuário, e se detecta a si mesmo executando em telefones com jailbreak ou root e reage a essas situações.
- **O Sistema Operacional:** Os sistemas operacionais e versões de SO em que o aplicativo executa (incluindo restrições de versão Android ou iOS), se o aplicativo é esperado executar em dispositivos que têm controles de Mobile Device Management (MDM), e vulnerabilidades relevantes do SO.
- **Rede:** Uso de protocolos de transporte seguros (ex., TLS), uso de chaves fortes e algoritmos criptográficos (ex., SHA-2) para proteger criptografia de tráfego de rede, uso de certificate pinning para verificar o endpoint, etc.
- **Serviços Remotos:** Os serviços remotos que o aplicativo consome e se seu comprometimento poderia comprometer o cliente.

#### Mapeamento do Aplicativo

Uma vez que o testador de segurança tem informação sobre o aplicativo e seu contexto, o próximo passo é mapear a estrutura e conteúdo do aplicativo, ex., identificando seus pontos de entrada, recursos e dados.

Quando teste de penetração é realizado em um paradigma white-box ou grey-box, quaisquer documentos do interior do projeto (diagramas de arquitetura, especificações funcionais, código, etc.) podem facilitar grandemente o processo. Se código-fonte está disponível, o uso de ferramentas SAST pode revelar informação valiosa sobre vulnerabilidades (ex., SQL Injection).
Ferramentas DAST podem suportar teste black-box e escanear automaticamente o aplicativo: enquanto um testador precisará de horas ou dias, um scanner pode realizar a mesma tarefa em alguns minutos. No entanto, é importante lembrar que ferramentas automáticas têm limitações e só encontrarão o que foram programadas para encontrar. Portanto, análise humana pode ser necessária para aumentar resultados de ferramentas automáticas (intuição é frequentemente chave para teste de segurança).

Modelagem de Ameaças é um artefato importante: documentos do workshop geralmente suportam muito a identificação de muita da informação que um testador de segurança precisa (pontos de entrada, ativos, vulnerabilidades, severidade, etc.). Testadores são fortemente aconselhados a discutir a disponibilidade de tais documentos com o cliente. Modelagem de ameaças deve ser uma parte chave do ciclo de vida de desenvolvimento de software. Geralmente ocorre nas fases iniciais de um projeto.

As [diretrizes de modelagem de ameaças definidas na OWASP](https://owasp.org/www-community/Threat_Modeling "OWASP Threat Modeling") são geralmente aplicáveis a aplicativos móveis.

#### Exploração

Infelizmente, restrições de tempo ou financeiras limitam muitos pentests ao mapeamento de aplicativo via scanners automatizados (para análise de vulnerabilidade, por exemplo). Embora vulnerabilidades identificadas durante a fase anterior possam ser interessantes, sua relevância deve ser confirmada com respeito a cinco eixos:

- **Potencial de dano** - o dano que pode resultar da exploração da vulnerabilidade
- **Reprodutibilidade** - facilidade de reproduzir o ataque
- **Explorabilidade** - facilidade de executar o ataque
- **Usuários afetados** - o número de usuários afetados pelo ataque
- **Descobribilidade** - facilidade de descobrir a vulnerabilidade

Contra todas as probabilidades, algumas vulnerabilidades podem não ser exploráveis e podem levar a comprometimentos menores, se houver. Outras vulnerabilidades podem parecer inofensivas à primeira vista, mas serem determinadas muito perigosas sob condições realistas de teste. Testadores que cuidadosamente passam pela fase de exploração suportam pentesting caracterizando vulnerabilidades e seus efeitos.

#### Relatório

Os achados do testador de segurança serão valiosos para o cliente apenas se estiverem claramente documentados. Um bom relatório de pentest deve incluir informação como, mas não limitada a, o seguinte:

- um resumo executivo
- uma descrição do escopo e contexto (ex., sistemas alvo)
- métodos usados
- fontes de informação (ou fornecidas pelo cliente ou descobertas durante o pentest)
- achados priorizados (ex., vulnerabilidades que foram estruturadas por classificação DREAD)
- achados detalhados
- recomendações para corrigir cada defeito

Muitos templates de relatório de pentest estão disponíveis na Internet: Google é seu amigo!

## Teste de Segurança e o SDLC

Embora os princípios de teste de segurança não tenham mudado fundamentalmente na história recente, técnicas de desenvolvimento de software mudaram dramaticamente. Enquanto a adoção generalizada de práticas Ágeis estava acelerando o desenvolvimento de software, testadores de segurança tiveram que se tornar mais rápidos e ágeis enquanto continuavam a entregar software confiável.

A seção seguinte é focada nesta evolução e descreve teste de segurança contemporâneo.

### Teste de Segurança durante o Ciclo de Vida de Desenvolvimento de Software

Desenvolvimento de software não é muito antigo, afinal, então o fim de desenvolver sem um framework é fácil de observar. Todos nós experimentamos a necessidade de um conjunto mínimo de regras para controlar o trabalho conforme o código-fonte cresce.

No passado, metodologias "Waterfall" eram as mais amplamente adotadas: desenvolvimento prosseguia por passos que tinham uma sequência predefinida. Limitado a um único passo, capacidade de retrocesso era uma desvantagem séria das metodologias Waterfall. Embora tenham características positivas importantes (fornecendo estrutura, ajudando testadores a clarificar onde esforço é necessário, sendo claras e fáceis de entender, etc.), elas também têm negativas (criando silos, sendo lentas, equipes especializadas, etc.).

Conforme o desenvolvimento de software amadureceu, a competição aumentou e desenvolvedores precisaram reagir a mudanças de mercado mais rapidamente enquanto criavam produtos de software com orçamentos menores. A ideia de menos estrutura tornou-se popular, e equipes menores colaboraram, quebrando silos por toda a organização. O conceito "Ágil" nasceu (Scrum, XP e RAD são exemplos bem conhecidos de implementações Ágeis); ele permitiu que equipes mais autônomas trabalhassem juntas mais rapidamente.

Segurança não era originalmente uma parte integral do desenvolvimento de software. Era uma reflexão tardia, realizada no nível de rede por equipes de operação que tinham que compensar por segurança pobre de software! Embora segurança não integrada fosse possível quando programas de software estavam localizados dentro de um perímetro, o conceito tornou-se obsoleto conforme novos tipos de consumo de software emergiram com tecnologias web, móveis e IoT. Hoje em dia, segurança deve ser cozida **dentro** do software porque compensar por vulnerabilidades é frequentemente muito difícil.

> "SDLC" será usado alternadamente com "Secure SDLC" na seção seguinte para ajudá-lo a internalizar a ideia que segurança é uma parte dos processos de desenvolvimento de software. No mesmo espírito, usamos o nome DevSecOps para enfatizar o fato que segurança é parte do DevOps.

### Visão Geral do SDLC

#### Descrição Geral do SDLC

SDLCs sempre consistem dos mesmos passos (o processo geral é sequencial no paradigma Waterfall e iterativo no paradigma Ágil):

- Realize uma **avaliação de risco** para o aplicativo e seus componentes para identificar seus perfis de risco. Esses perfis de risco tipicamente dependem do apetite de risco da organização e requisitos regulatórios aplicáveis. A avaliação de risco também é baseada em fatores, incluindo se o aplicativo é acessível via Internet e o tipo de dados que o aplicativo processa e armazena. Todos os tipos de riscos devem ser considerados: financeiros, de marketing, industriais, etc. Políticas de classificação de dados especificam quais dados são sensíveis e como devem ser protegidos.
- **Requisitos de Segurança** são determinados no início de um projeto ou ciclo de desenvolvimento, quando requisitos funcionais estão sendo coletados. **Casos de Abuso** são adicionados conforme casos de uso são criados. Equipes (incluindo equipes de desenvolvimento) podem receber treinamento de segurança (como Codificação Segura) se precisarem.
Você pode usar o [OWASP MASVS](https://mas.owasp.org/MASVS/ "OWASP MASVS") e [OWASP MASWE](https://mas.owasp.org/MASWE/ "OWASP MASWE") para determinar os requisitos de segurança de aplicativos móveis com base na fase de avaliação de risco. Revisar iterativamente requisitos quando funcionalidades e classes de dados são adicionadas é comum, especialmente com projetos Ágeis.
- **Modelagem de Ameaças**, que é basicamente a identificação, enumeração, priorização e tratamento inicial de ameaças, é um artefato fundamental que deve ser realizado conforme arquitetura e design progridem. **Arquitetura de Segurança**, um fator de Modelagem de Ameaças, pode ser refinada (tanto para aspectos de software quanto hardware) após a fase de Modelagem de Ameaças. **Regras de Codificação Segura** são estabelecidas e a lista de **Ferramentas de Segurança** que serão usadas é criada. A estratégia para **Teste de Segurança** é clarificada.
- Todos os requisitos de segurança e considerações de design devem ser armazenados no sistema de Gerenciamento de Ciclo de Vida de Aplicação (ALM) (também conhecido como rastreador de issues) que a equipe de desenvolvimento/ops usa para garantir integração apertada de requisitos de segurança no fluxo de trabalho de desenvolvimento. Os requisitos de segurança devem conter trechos de código-fonte relevantes para que desenvolvedores possam rapidamente referenciar os trechos. Criar um repositório dedicado que está sob controle de versão e contém apenas esses trechos de código é uma estratégia de codificação segura que é mais benéfica que a abordagem tradicional (armazenar as diretrizes em documentos word ou PDFs).
- **Desenvolva o software com segurança**. Para aumentar a segurança do código, você deve completar atividades como **Revisões de Código de Segurança**, **Static Application Security Testing** e **Teste de Unidade de Segurança**. Embora análogos de qualidade dessas atividades de segurança existam, a mesma lógica deve ser aplicada à segurança, ex., revisando, analisando e testando código para defeitos de segurança (por exemplo, validação de entrada ausente, falha em liberar todos os recursos, etc.).
- Em seguida vem o tão aguardado teste de candidato a release: tanto **Teste de Penetração** manual quanto automatizado ("Pentests"). **Dynamic Application Security Testing** é geralmente realizado durante esta fase também.
- Após o software ter sido **Credenciado** durante **Aceitação** por todas as partes interessadas, pode ser seguramente transicionado para equipes de **Operação** e colocado em Produção.
- A última fase, muito negligenciada, é o **Descomissionamento** seguro do software após seu fim de uso.

A imagem abaixo ilustra todas as fases e artefatos:

<img src="Images/Chapters/0x04b/SDLCOverview.jpg" width="100%" />

Com base no perfil de risco geral do projeto, você pode simplificar (ou até pular) alguns artefatos, e pode adicionar outros (aprovações intermediárias formais, documentação formal de certos pontos, etc.). **Sempre lembre duas coisas: um SDLC é destinado a reduzir riscos associados ao desenvolvimento de software, e é um framework que ajuda você a configurar controles para esse fim.** Esta é uma descrição genérica de SDLC; sempre adapte este framework aos seus projetos.

#### Definindo uma Estratégia de Teste

Estratégias de teste especificam os testes que serão realizados durante o SDLC, bem como a frequência de teste. Estratégias de teste são usadas para ter certeza que o produto de software final atende objetivos de segurança, que são geralmente determinados por equipes legais/marketing/corporativas dos clientes.
A estratégia de teste é geralmente criada durante a fase de Design Seguro, após os riscos terem sido clarificados (durante a fase de Iniciação) e antes do desenvolvimento de código (a fase de Implementação Segura) começar. A estratégia requer entrada de atividades como Gerenciamento de Risco, Modelagem de Ameaças anterior e Engenharia de Segurança.

Uma Estratégia de Teste não precisa ser formalmente escrita: pode ser descrita através de Histórias (em projetos Ágeis), rapidamente enumerada em listas de verificação, ou especificada como casos de teste para uma determinada ferramenta. No entanto, a estratégia deve definitivamente ser compartilhada porque deve ser implementada por uma equipe diferente da equipe que a definiu. Além disso, todas as equipes técnicas devem concordar com ela para garantir que não coloque encargos inaceitáveis em nenhuma delas.

Estratégias de Teste abordam tópicos como os seguintes:

- objetivos e descrições de risco
- planos para atender objetivos, redução de risco, quais testes serão obrigatórios, quem os realizará, como e quando serão realizados
- critérios de aceitação

Para acompanhar o progresso e efetividade da estratégia de teste, métricas devem ser definidas, continuamente atualizadas durante o projeto e periodicamente comunicadas. Um livro inteiro poderia ser escrito sobre escolher métricas relevantes; o máximo que podemos dizer aqui é que elas dependem de perfis de risco, projetos e organizações. Exemplos de métricas incluem o seguinte:

- o número de histórias relacionadas a controles de segurança que foram implementadas com sucesso
- cobertura de código para testes de unidade de controles de segurança e funcionalidades sensíveis
- o número de bugs de segurança encontrados para cada build via ferramentas de análise estática
- tendências em backlogs de bugs de segurança (que podem ser ordenados por urgência)

Estas são apenas sugestões; outras métricas podem ser mais relevantes para seu projeto. Métricas são ferramentas poderosas para colocar um projeto sob controle, desde que deem aos gerentes de projeto uma perspectiva clara e sintética do que está acontecendo e do que precisa ser melhorado.

Distinguir entre testes realizados por uma equipe interna e testes realizados por uma terceira parte independente é importante. Testes internos são geralmente úteis para melhorar operações diárias, enquanto testes de terceiras partes são mais benéficos para toda a organização. Testes internos podem ser realizados com bastante frequência, mas teste de terceira parte acontece no máximo uma ou duas vezes por ano; também, os primeiros são menos caros que os últimos.
Ambos são necessários, e muitos regulamentos mandatam testes de uma terceira parte independente porque tais testes podem ser mais confiáveis.

### Teste de Segurança em Waterfall

#### O que é Waterfall e Como Atividades de Teste São Organizadas

Basicamente, SDLC não manda o uso de qualquer ciclo de vida de desenvolvimento: é seguro dizer que segurança pode (e deve!) ser abordada em qualquer situação.

Metodologias Waterfall eram populares antes do século 21. A aplicação mais famosa é chamada de "modelo V", no qual fases são realizadas em sequência e você pode retroceder apenas um único passo.
As atividades de teste deste modelo ocorrem em sequência e são realizadas como um todo, principalmente no ponto do ciclo de vida quando a maior parte do desenvolvimento do aplicativo está completa. Esta sequência de atividade significa que mudar a arquitetura e outros fatores que foram configurados no início do projeto é dificilmente possível mesmo que código possa ser mudado após defeitos terem sido identificados.

### Teste de Segurança para Agile/DevOps e DevSecOps

DevOps refere-se a práticas que focam em uma colaboração próxima entre todas as partes interessadas envolvidas no desenvolvimento de software (
geralmente chamados Devs) e operações (geralmente chamados Ops). DevOps não é sobre fundir Devs e Ops.
Desenvolvimento e equipes de operações originalmente trabalhavam em silos, quando empurrar software desenvolvido para produção poderia levar uma quantidade significativa de tempo. Quando equipes de desenvolvimento tornaram mover mais entregas para produção necessário trabalhando com Agile, equipes de operação tiveram que acelerar para acompanhar o ritmo. DevOps é a evolução necessária da solução para esse desafio na medida em que permite que software seja liberado para usuários mais rapidamente. Isso é amplamente realizado via automação extensiva de build, o processo de testar e liberar software, e mudanças de infraestrutura (além do aspecto de colaboração do DevOps). Esta automação é incorporada no pipeline de deployment com os conceitos de Integração Contínua e Entrega Contínua (CI/CD).

Pessoas podem assumir que o termo "DevOps" representa colaboração entre equipes de desenvolvimento e operações apenas, no entanto, como o líder de pensamento DevOps Gene Kim coloca: "À primeira vista, parece que os problemas são apenas entre Devs e Ops, mas teste está lá, e você tem objetivos de segurança da informação, e a necessidade de proteger sistemas e dados. Estas são preocupações de alto nível da gerência, e elas se tornaram parte do quadro DevOps."

Em outras palavras, colaboração DevOps inclui equipes de qualidade, equipes de segurança e muitas outras equipes relacionadas ao projeto. Quando você ouve "DevOps" hoje, você provavelmente deveria estar pensando em algo como [DevOpsQATestInfoSec](https://techbeacon.com/evolution-devops-new-thinking-gene-kim "The evolution of DevOps: Gene Kim on getting to continuous delivery"). De fato, valores DevOps pertencem a aumentar não apenas velocidade mas também qualidade, segurança, confiabilidade, estabilidade e resiliência.

Segurança é tão crítica para o sucesso do negócio quanto a qualidade geral, performance e usabilidade de um aplicativo. Como ciclos de desenvolvimento são encurtados e frequências de entrega aumentadas, ter certeza que qualidade e segurança são construídas desde o início se torna essencial. **DevSecOps** é tudo sobre adicionar segurança aos processos DevOps. A maioria dos defeitos é identificada durante produção. DevOps especifica melhores práticas para identificar tantos defeitos quanto possível cedo no ciclo de vida e para minimizar o número de defeitos no aplicativo liberado.

No entanto, DevSecOps não é apenas um processo linear orientado a entregar o melhor software possível para operações; é também um mandato que operações monitorem de perto software que está em produção para identificar issues e corrigi-los formando um loop de feedback rápido e eficiente com desenvolvimento. DevSecOps é um processo através do qual Melhoria Contínua é fortemente enfatizada.

<img src="Images/Chapters/0x04b/DevSecOpsProcess.JPG" width="100%" />

O aspecto humano desta ênfase é refletido na criação de equipes multifuncionais que trabalham juntas para alcançar resultados de negócios. Esta seção é focada em interações necessárias e integrando segurança no ciclo de vida de desenvolvimento (que começa com início do projeto e termina com a entrega de valor para usuários).

#### O que são Agile e DevSecOps e Como Atividades de Teste São Organizadas

##### Visão Geral

Automação é uma prática chave DevSecOps: como afirmado anteriormente, a frequência de entregas de desenvolvimento para operação aumenta quando comparada à abordagem tradicional, e atividades que geralmente requerem tempo precisam acompanhar, ex. entregar o mesmo valor agregado enquanto levando menos tempo. Atividades improdutivas devem consequentemente ser abandonadas, e tarefas essenciais devem ser aceleradas. Essas mudanças impactam mudanças de infraestrutura, deployment e segurança:

- infraestrutura está sendo implementada como **Infrastructure as Code**
- deployment está se tornando mais scriptado, traduzido através dos conceitos de **Integração Contínua** e **Entrega Contínua**
- **atividades de segurança** estão sendo automatizadas tanto quanto possível e ocorrendo por todo o ciclo de vida

As seções seguintes fornecem mais detalhes sobre esses três pontos.

##### Infrastructure as Code

Em vez de provisionar manualmente recursos de computação (servidores físicos, máquinas virtuais, etc.) e modificar arquivos de configuração, Infrastructure as Code é baseado no uso de ferramentas e automação para acelerar o processo de provisionamento e torná-lo mais confiável e repetível. Scripts correspondentes são frequentemente armazenados sob controle de versão para facilitar compartilhamento e resolução de issues.

Práticas de Infrastructure as Code facilitam colaboração entre equipes de desenvolvimento e operações, com os seguintes resultados:

- Devs entendem melhor infraestrutura de um ponto de vista familiar e podem preparar recursos que o aplicativo em execução requererá.
- Ops operam um ambiente que melhor se adequa ao aplicativo, e eles compartilham uma linguagem com Devs.

Infrastructure as Code também facilita a construção dos ambientes requeridos por projetos clássicos de criação de software, para **desenvolvimento** ("DEV"), **integração** ("INT"), **teste** ("PPR" para Pré-Produção. Alguns testes são geralmente realizados em ambientes anteriores, e testes PPR pertencem principalmente a não-regressão e performance com dados que são similares a dados usados em produção), e **produção** ("PRD"). O valor de infrastructure as code reside na possível similaridade entre ambientes (eles deveriam ser os mesmos).

Infrastructure as Code é comumente usado para projetos que têm recursos baseados em Cloud porque muitos fornecedores fornecem APIs que podem ser usadas para provisionar itens (como máquinas virtuais, espaços de armazenamento, etc.) e trabalhar em configurações (ex., modificando tamanhos de memória ou o número de CPUs usadas por máquinas virtuais). Essas APIs fornecem alternativas a administradores realizarem essas atividades a partir de consoles de monitoramento.

As principais ferramentas neste domínio são [Puppet](https://puppet.com/ "Puppet"), [Terraform](https://www.terraform.io/ "Terraform"), [Packer](https://www.packer.io/ "Packer"), [Chef](https://www.chef.io/chef/ "Chef") e [Ansible](https://www.ansible.com/ "Ansible").

##### Deployment

A sofisticação do pipeline de deployment depende da maturidade da organização do projeto ou equipe de desenvolvimento. Em sua forma mais simples, o pipeline de deployment consiste de uma fase de commit. A fase de commit geralmente envolve executar verificações simples de compilador e o conjunto de testes de unidade, bem como criar um artefato implantável do aplicativo. Um candidato a release é a última versão que foi checkada no trunk do sistema de controle de versão. Candidatos a release são avaliados pelo pipeline de deployment para conformidade com padrões que devem cumprir para deployment em produção.

A fase de commit é projetada para fornecer feedback instantâneo para desenvolvedores e é portanto executada em cada commit para o trunk. Restrições de tempo existem por causa desta frequência. A fase de commit geralmente deve estar completa dentro de cinco minutos, e não deveria levar mais que dez. Aderir a esta restrição de tempo é bastante desafiador quando se trata de segurança porque muitas ferramentas de segurança não podem ser executadas rapidamente o suficiente (#paul, #mcgraw).

CI/CD significa "Integração Contínua/Entrega Contínua" em alguns contextos e "Integração Contínua/Implantação Contínua" em outros. Na verdade, a lógica é:

- Ações de build de Integração Contínua (seja acionadas por um commit ou realizadas regularmente) usam todo o código-fonte para construir um candidato a release. Testes podem então ser realizados e a conformidade do release com regras de segurança, qualidade, etc., pode ser verificada. Se a conformidade for confirmada, o processo pode continuar; caso contrário, a equipe de desenvolvimento deve remediar o(s) issue(s) e propor mudanças.
- Candidatos a Entrega Contínua podem prosseguir para o ambiente de pré-produção. Se o release puder então ser validado (seja manual ou automaticamente), o deployment pode continuar. Se não, a equipe do projeto será notificada e ação(ões) apropriada(s) deve(m) ser tomada(s).
- Releases de Implantação Contínua são transicionados diretamente da integração para produção, ex., eles se tornam acessíveis ao usuário. No entanto, nenhum release deve ir para produção se defeitos significativos tiverem sido identificados durante atividades anteriores.

A entrega e implantação de aplicativos com baixa ou média sensibilidade podem ser fundidas em um único passo, e validação pode ser realizada após a entrega. No entanto, manter essas duas ações separadas e usar validação forte é fortemente aconselhado para aplicativos sensíveis.

##### Segurança

Neste ponto, a grande questão é: agora que outras atividades requeridas para entregar código são completadas significativamente mais rápido e mais efetivamente, como a segurança pode acompanhar? Como podemos manter um nível apropriado de segurança? Entregar valor para usuários mais frequentemente com segurança diminuída definitivamente não seria bom!

Mais uma vez, a resposta é automação e ferramentas: implementando esses dois conceitos por todo o ciclo de vida do projeto, você pode manter e melhorar a segurança. Quanto maior o nível esperado de segurança, mais controles, pontos de verificação e ênfase ocorrerão. Os seguintes são exemplos:

- Static Application Security Testing pode ocorrer durante a fase de desenvolvimento, e pode ser integrado no processo de Integração Contínua com mais ou menos ênfase nos resultados do scan. Você pode estabelecer Regras de Codificação Segura mais ou menos exigentes e usar ferramentas SAST para verificar a efetividade de sua implementação.
- Dynamic Application Security Testing pode ser automaticamente realizado após o aplicativo ter sido construído (ex., após Integração Contínua ter ocorrido) e antes da entrega, novamente, com mais ou menos ênfase nos resultados.
- Você pode adicionar pontos de verificação de validação manual entre fases consecutivas, por exemplo, entre entrega e implantação.

A segurança de um aplicativo desenvolvido com DevOps deve ser considerada durante operações. Os seguintes são exemplos:

- Scanning deve ocorrer regularmente (tanto no nível de infraestrutura quanto de aplicativo).
- Pentesting pode ocorrer regularmente. (A versão do aplicativo usada em produção é a versão que deve ser pentestada, e o teste deve ocorrer em um ambiente dedicado e incluir dados que são similares aos dados da versão de produção. Veja a seção sobre Teste de Penetração para mais detalhes.)
- Monitoramento ativo deve ser realizado para identificar issues e remediá-los o mais rápido possível via o loop de feedback.

<img src="Images/Chapters/0x04b/ExampleOfADevSecOpsProcess.jpg" width="100%" />

## Referências

- [paul] - M. Paul. Official (ISC)2 Guide to the CSSLP CBK, Second Edition ((ISC)2 Press), 2014
- [mcgraw] - G McGraw. Software Security: Building Security In, 2006