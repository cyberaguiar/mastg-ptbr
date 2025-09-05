# Introdução ao Projeto OWASP Mobile Application Security

Novas tecnologias sempre trazem novos riscos de segurança, e as preocupações com apps móveis diferem das de softwares tradicionais de desktop em aspectos importantes. Embora os sistemas operacionais móveis modernos tendam a ser mais seguros do que os sistemas operacionais de desktop, problemas ainda podem surgir se desenvolvedores não considerarem a segurança com cuidado durante o desenvolvimento do app. Esses riscos de segurança frequentemente vão além das preocupações usuais com armazenamento de dados, comunicação entre apps, uso adequado de APIs criptográficas e comunicação de rede segura.

## Como usar o Mobile Application Security Project

Primeiro, o Projeto recomenda que sua estratégia de segurança para apps móveis seja baseada no [OWASP Mobile Application Security _Verification Standard_ (MASVS)](https://mas.owasp.org/MASVS/), que define um modelo de segurança para apps móveis e lista requisitos genéricos de segurança para apps. O MASVS foi projetado para ser usado por arquitetos, desenvolvedores, testadores, profissionais de segurança e consumidores para definir e entender as qualidades de um app móvel seguro. Depois de determinar como o OWASP MASVS se aplica ao modelo de segurança do seu app, o Projeto sugere o uso do [OWASP Mobile Application Security _Testing Guide_ (MASTG)](https://mas.owasp.org/MASTG/). O Testing Guide mapeia o mesmo conjunto básico de requisitos de segurança oferecidos pelo MASVS e, dependendo do contexto, eles podem ser usados individualmente ou combinados para atingir diferentes objetivos.

<img src="Images/Chapters/0x03/owasp-mobile-overview.png" width="50%" />

Por exemplo, os requisitos do MASVS podem ser usados nas etapas de planejamento e design de arquitetura de um app, enquanto a checklist e o guia de testes podem servir como base para testes manuais de segurança ou como modelo para testes automatizados durante ou após o desenvolvimento. No capítulo ["Testes de Segurança de Aplicativos Móveis"](0x04b-Mobile-App-Security-Testing.md) descreveremos como aplicar a checklist e o MASTG em um teste de penetração de app móvel.

## O que o Mobile Testing Guide cobre

Ao longo deste guia, focaremos em apps para Android e iOS executados em smartphones. Essas plataformas dominam o mercado atualmente e também são usadas em outras classes de dispositivos, incluindo tablets, smartwatches, smart TVs, unidades de infoentretenimento automotivo e outros sistemas embarcados. Mesmo que essas classes adicionais estejam fora de escopo, ainda é possível aplicar a maior parte do conhecimento e das técnicas de teste descritas neste guia, com algumas adaptações dependendo do dispositivo alvo.

Dada a vasta quantidade de frameworks para apps móveis, seria impossível cobrir todos exaustivamente. Portanto, focamos em apps _nativos_ de cada sistema operacional. No entanto, as mesmas técnicas também são úteis ao lidar com apps web ou híbridos (no final das contas, independentemente do framework, todo app é baseado em componentes nativos).

## Navegando pelo OWASP MASTG

O MASTG contém descrições de todos os requisitos especificados no MASVS. O MASTG contém as seguintes seções principais:

1. O [General Testing Guide](0x04a-Mobile-App-Taxonomy.md) apresenta uma metodologia de testes de segurança para apps móveis e técnicas gerais de análise de vulnerabilidades aplicadas à segurança de apps móveis. Ele também inclui casos de teste técnicos adicionais independentes de sistema operacional, como autenticação e gerenciamento de sessão, comunicações de rede e criptografia.

2. O [Android Testing Guide](0x05a-Platform-Overview.md) aborda testes de segurança móvel para a plataforma Android, incluindo fundamentos de segurança, casos de teste, técnicas de engenharia reversa e medidas de prevenção, bem como técnicas e mitigação de adulteração.

3. O [iOS Testing Guide](0x06a-Platform-Overview.md) aborda testes de segurança móvel para a plataforma iOS, incluindo uma visão geral do sistema operacional, testes de segurança, técnicas de engenharia reversa e prevenção, além de técnicas e mitigação de adulteração.

## Como profissionais de segurança devem tratar os testes de segurança móvel

Muitos pentesters de apps móveis têm experiência prévia em testes de penetração de rede e de apps web, uma qualidade valiosa para testes de apps móveis. Quase todo app móvel conversa com um serviço de backend, e esses serviços estão sujeitos aos mesmos tipos de ataques que conhecemos em apps web em desktops. Apps móveis possuem uma superfície de ataque menor e, portanto, oferecem mais segurança contra injeção e ataques semelhantes. Em vez disso, o MASTG prioriza a proteção de dados no dispositivo e na rede para aumentar a segurança móvel.

## Visão geral do OWASP MASVS: áreas-chave na segurança de apps móveis

### MASVS-STORAGE: Armazenamento de dados e privacidade

O Standard baseia-se no princípio de que proteger dados sensíveis, como credenciais e informações privadas, é crucial para a segurança móvel. Se um app não usa corretamente as APIs do sistema operacional, especialmente aquelas que lidam com armazenamento local ou comunicação entre processos (IPC), ele pode expor dados sensíveis a outros apps no mesmo dispositivo ou vazar informações para armazenamento em nuvem, backups ou cache do teclado. E como dispositivos móveis têm maior probabilidade de serem perdidos ou roubados, invasores podem obter acesso físico ao aparelho, facilitando a recuperação dos dados.

Portanto, é necessário ter cuidado extra ao proteger os dados armazenados de usuários em apps móveis. Algumas soluções incluem o uso adequado de APIs de armazenamento de chaves e o uso de recursos de segurança baseados em hardware (quando disponíveis).

A fragmentação é um problema especialmente em dispositivos Android. Nem todos os aparelhos oferecem armazenamento seguro baseado em hardware, e muitos executam versões desatualizadas do Android. Para que um app seja suportado nesses dispositivos, ele teria que ser criado com uma versão antiga da API do Android, que pode não incluir recursos de segurança importantes. Para máxima segurança, a melhor escolha é criar apps com a versão de API atual, mesmo que isso exclua alguns usuários.

### MASVS-CRYPTO: Criptografia

Criptografia é um componente essencial na proteção de dados armazenados em um dispositivo móvel. Também é uma área onde as coisas podem dar muito errado, especialmente quando convenções padrão não são seguidas. É fundamental garantir que o app use criptografia conforme as melhores práticas da indústria, incluindo uso de bibliotecas criptográficas comprovadas, escolha e configuração adequadas de primitivas criptográficas e um gerador de números aleatórios apropriado sempre que a aleatoriedade for necessária.

### MASVS-AUTH: Autenticação e autorização

Na maioria dos casos, fazer o usuário autenticar-se em um serviço remoto é parte integrante da arquitetura geral de um app móvel. Embora a maior parte da lógica de autenticação e autorização ocorra no endpoint, também existem desafios de implementação no lado do app. Ao contrário de apps web, apps móveis frequentemente armazenam tokens de sessão de longa duração que são desbloqueados com recursos de autenticação do dispositivo, como leitura de impressão digital. Isso permite um login mais rápido e melhor experiência do usuário (ninguém gosta de digitar senhas complexas), mas também introduz complexidade adicional e margem para erros.

Arquiteturas de apps móveis também incorporam cada vez mais frameworks de autorização (como OAuth2) que delegam a autenticação a um serviço separado ou terceirizam o processo para um provedor de autenticação. O uso de OAuth2 permite que a lógica de autenticação do lado do cliente seja terceirizada para outros apps no mesmo dispositivo (por exemplo, o navegador do sistema). Testadores de segurança devem conhecer as vantagens e desvantagens dos diferentes frameworks e arquiteturas de autorização.

### MASVS-NETWORK: Comunicação de rede

Dispositivos móveis conectam-se regularmente a diversas redes, incluindo redes Wi-Fi públicas compartilhadas com outros clientes (potencialmente maliciosos). Isso cria oportunidades para uma grande variedade de ataques baseados em rede, que vão de simples a complexos, dos antigos aos recentes. É crucial manter a confidencialidade e integridade das informações trocadas entre o app móvel e serviços remotos. Como requisito básico, apps móveis devem estabelecer um canal seguro e criptografado para comunicação de rede usando o protocolo TLS com configurações apropriadas.

### MASVS-PLATFORM: Interação com a plataforma móvel

Arquiteturas de sistemas operacionais móveis diferem significativamente das arquiteturas de desktop clássicas. Por exemplo, todos os sistemas operacionais móveis implementam sistemas de permissões que regulam o acesso a APIs específicas. Eles também oferecem mais (Android) ou menos (iOS) recursos de comunicação entre processos (IPC) que permitem que apps troquem sinais e dados. Esses recursos específicos da plataforma trazem suas próprias armadilhas. Por exemplo, se APIs de IPC forem mal utilizadas, dados ou funcionalidades sensíveis podem ser expostos inadvertidamente a outros apps no dispositivo.

### MASVS-CODE: Qualidade do código e mitigação de exploits

Questões tradicionais de injeção e gerenciamento de memória raramente são vistas em apps móveis devido à superfície de ataque reduzida. Apps móveis interagem principalmente com o serviço de backend confiável e a interface do usuário, de modo que, mesmo que existam vulnerabilidades de estouro de buffer, elas geralmente não abrem vetores de ataque úteis. O mesmo se aplica a exploits de navegador como cross-site scripting (XSS, que permite a injeção de scripts em páginas web), comuns em apps web. Entretanto, sempre há exceções. XSS é teoricamente possível no ambiente móvel em alguns casos, mas é muito raro ver um problema de XSS explorável por um indivíduo.

Essa proteção contra problemas de injeção e gerenciamento de memória não significa que desenvolvedores possam se dar ao luxo de escrever código descuidado. Seguir melhores práticas de segurança resulta em builds de release endurecidos (seguros) que são resilientes contra adulteração. Recursos de segurança gratuitos oferecidos por compiladores e SDKs móveis ajudam a aumentar a segurança e mitigar ataques.

### MASVS-RESILIENCE: Antitampering e antirreversing

Há três coisas que você nunca deve comentar em conversas educadas: religião, política e ofuscação de código. Muitos especialistas em segurança descartam proteções do lado do cliente prontamente. Contudo, controles de proteção de software são amplamente usados no mundo de apps móveis, então testadores de segurança precisam lidar com essas proteções. Acreditamos que há benefícios em proteções do lado do cliente quando empregadas com um propósito claro e expectativas realistas em mente e não como substituição de controles de segurança.
