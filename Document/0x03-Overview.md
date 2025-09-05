# Introdução ao Projeto OWASP Mobile Application Security

Nova tecnologia sempre introduz novos riscos de segurança, e as preocupações de segurança para aplicativos móveis diferem de software desktop tradicional de maneiras importantes. Embora sistemas operacionais móveis modernos tendam a ser mais seguros que sistemas operacionais desktop tradicionais, problemas ainda podem aparecer se desenvolvedores não considerarem cuidadosamente a segurança durante o desenvolvimento de aplicativos móveis. Esses riscos de segurança frequentemente vão além das preocupações usuais com armazenamento de dados, comunicação entre aplicativos, uso adequado de APIs criptográficas e comunicação segura de rede.

## Como Usar o Projeto Mobile Application Security

Primeiro, o Projeto recomenda que suas estratégias de segurança de aplicativos móveis devem ser baseadas no [OWASP Mobile Application Security _Verification Standard_ (MASVS)](https://mas.owasp.org/MASVS/), que define um modelo de segurança de aplicativos móveis e lista requisitos genéricos de segurança para aplicativos móveis. O MASVS é projetado para ser usado por arquitetos, desenvolvedores, testadores, profissionais de segurança e consumidores para definir e entender as qualidades de um aplicativo móvel seguro. Depois de determinar como o OWASP MASVS se aplica ao modelo de segurança do seu aplicativo móvel, o Projeto sugere que você use o [OWASP Mobile Application Security _Testing Guide_ (MASTG)](https://mas.owasp.org/MASTG/). O Guia de Teste mapeia para o mesmo conjunto básico de requisitos de segurança oferecidos pelo MASVS e dependendo do contexto, eles podem ser usados individualmente ou combinados para alcançar objetivos diferentes.

<img src="Images/Chapters/0x03/owasp-mobile-overview.png" width="50%" />

Por exemplo, os requisitos do MASVS podem ser usados nos estágios de planejamento e design de arquitetura de um aplicativo, enquanto a lista de verificação e o guia de teste podem servir como base para testes de segurança manuais ou como template para testes de segurança automatizados durante ou após o desenvolvimento. No capítulo ["Mobile App Security Testing"](0x04b-Mobile-App-Security-Testing_PT-BR.md) descreveremos como você pode aplicar a lista de verificação e o MASTG a um teste de penetração de aplicativo móvel.

## O que é Abordado no Guia de Teste Móvel

Ao longo deste guia, focaremos em aplicativos para Android e iOS executando em smartphones. Essas plataformas atualmente dominam o mercado e também executam em outras classes de dispositivos, incluindo tablets, smartwatches, smart TVs, unidades de infotainment automotivo e outros sistemas embarcados. Mesmo que essas classes adicionais de dispositivos estejam fora do escopo, você ainda pode aplicar a maior parte do conhecimento e técnicas de teste descritas neste guia com algum desvio dependendo do dispositivo alvo.

Dada a vasta quantidade de frameworks de aplicativos móveis disponíveis, seria impossível cobrir todos exaustivamente. Portanto, focamos em aplicativos _nativos_ em cada sistema operacional. No entanto, as mesmas técnicas também são úteis ao lidar com aplicativos web ou híbridos (em última análise, não importa o framework, todo aplicativo é baseado em componentes nativos).

## Navegando pelo OWASP MASTG

O MASTG contém descrições de todos os requisitos especificados no MASVS. O MASTG contém as seguintes seções principais:

1. O [Guia de Teste Geral](0x04a-Mobile-App-Taxonomy_PT-BR.md) contém uma metodologia de teste de segurança de aplicativos móveis e técnicas gerais de análise de vulnerabilidade conforme se aplicam à segurança de aplicativos móveis. Também contém casos de teste técnicos adicionais que são independentes de SO, como autenticação e gerenciamento de sessão, comunicações de rede e criptografia.

2. O [Guia de Teste Android](0x05a-Platform-Overview_PT-BR.md) cobre teste de segurança móvel para a plataforma Android, incluindo conceitos básicos de segurança, casos de teste de segurança, técnicas de engenharia reversa e prevenção, e técnicas de adulteração e prevenção.

3. O [Guia de Teste iOS](0x06a-Platform-Overview_PT-BR.md) cobre teste de segurança móvel para a plataforma iOS, incluindo uma visão geral do SO iOS, teste de segurança, técnicas de engenharia reversa e prevenção, e técnicas de adulteração e prevenção.

## Como Pessoal de Segurança Deve Abordar o Teste de Segurança Móvel

Muitos testadores de penetração de aplicativos móveis têm formação em teste de penetração de rede e aplicativos web, uma qualidade que é valiosa para teste de aplicativos móveis. Quase todo aplicativo móvel se comunica com um serviço backend, e esses serviços são propensos aos mesmos tipos de ataques com os quais estamos familiarizados em aplicativos web em máquinas desktop. Aplicativos móveis têm uma superfície de ataque menor e, portanto, têm mais segurança contra injeção e ataques similares. Em vez disso, o MASTG prioriza a proteção de dados no dispositivo e na rede para aumentar a segurança móvel.

## Visão Geral do OWASP MASVS: Áreas-Chave em Segurança de Aplicativos Móveis

### MASVS-STORAGE: Armazenamento de Dados e Privacidade

O Standard é baseado no princípio de que proteger dados sensíveis, como credenciais de usuário e informações privadas, é crucial para a segurança móvel. Se um aplicativo não usar as APIs do sistema operacional adequadamente, especialmente aquelas que lidam com armazenamento local ou comunicação entre processos (IPC), o aplicativo pode expor dados sensíveis a outros aplicativos executando no mesmo dispositivo ou pode vazar dados não intencionalmente para armazenamento em nuvem, backups ou o cache do teclado. E como dispositivos móveis são mais propensos a serem perdidos ou roubados, atacantes podem realmente obter acesso físico ao dispositivo, o que tornaria mais fácil recuperar os dados.

Assim, devemos tomar cuidado extra para proteger dados de usuário armazenados em aplicativos móveis. Algumas soluções podem incluir APIs apropriadas de armazenamento de chaves e usar recursos de segurança com suporte de hardware (quando disponíveis).

Fragmentação é um problema que lidamos especialmente em dispositivos Android. Nem todo dispositivo Android oferece armazenamento seguro com suporte de hardware, e muitos dispositivos executam versões desatualizadas do Android. Para um aplicativo ser suportado nesses dispositivos desatualizados, ele teria que ser criado usando uma versão mais antiga da API do Android que pode carecer de recursos importantes de segurança. Para máxima segurança, a melhor escolha é criar aplicativos com a versão atual da API, mesmo que isso exclua alguns usuários.

### MASVS-CRYPTO: Criptografia

Criptografia é um ingrediente essencial quando se trata de proteger dados armazenados em um dispositivo móvel. Também é uma área onde as coisas podem dar horrivelmente errado, especialmente quando convenções padrão não são seguidas. É essencial garantir que a aplicação use criptografia de acordo com as melhores práticas da indústria, incluindo o uso de bibliotecas criptográficas comprovadas, uma escolha e configuração adequadas de primitivas criptográficas, bem como um gerador de números aleatórios adequado onde quer que aleatoriedade seja necessária.

### MASVS-AUTH: Autenticação e Autorização

Na maioria dos casos, enviar usuários para fazer login em um serviço remoto é uma parte integral da arquitetura geral do aplicativo móvel. Embora a maior parte da lógica de autenticação e autorização aconteça no endpoint, também existem alguns desafios de implementação no lado do aplicativo móvel. Diferente de aplicativos web, aplicativos móveis frequentemente armazenam tokens de sessão de longa duração que são desbloqueados com recursos de autenticação usuário-dispositivo, como digitalização de impressão digital. Embora isso permita um login mais rápido e melhor experiência do usuário (ninguém gosta de digitar senhas complexas), também introduz complexidade adicional e espaço para erro.

Arquiteturas de aplicativos móveis também incorporam cada vez mais frameworks de autorização (como OAuth2) que delegam autenticação para um serviço separado ou terceirizam o processo de autenticação para um provedor de autenticação. Usar OAuth2 permite que a lógica de autenticação do lado do cliente seja terceirizada para outros aplicativos no mesmo dispositivo (ex: o navegador do sistema). Testadores de segurança devem conhecer as vantagens e desvantagens de diferentes frameworks e arquiteturas de autorização possíveis.

### MASVS-NETWORK: Comunicação de Rede

Dispositivos móveis se conectam regularmente a uma variedade de redes, incluindo redes Wi-Fi públicas compartilhadas com outros clientes (potencialmente maliciosos). Isso cria oportunidades para uma ampla variedade de ataques baseados em rede, variando de simples a complicados e antigos a novos. É crucial manter a confidencialidade e integridade das informações trocadas entre o aplicativo móvel e endpoints de serviço remoto. Como requisito básico, aplicativos móveis devem configurar um canal seguro e criptografado para comunicação de rede usando o protocolo TLS com configurações apropriadas.

### MASVS-PLATFORM: Interação com a Plataforma Móvel

Arquiteturas de sistemas operacionais móveis diferem de arquiteturas desktop clássicas de maneiras importantes. Por exemplo, todos os sistemas operacionais móveis implementam sistemas de permissão de aplicativos que regulam o acesso a APIs específicas. Eles também oferecem instalações de comunicação entre processos (IPC) mais ricas (Android) ou menos ricas (iOS) que permitem que aplicativos troquem sinais e dados. Esses recursos específicos da plataforma vêm com seu próprio conjunto de armadilhas. Por exemplo, se APIs de IPC forem mal utilizadas, dados ou funcionalidade sensíveis podem ser expostos não intencionalmente a outros aplicativos executando no dispositivo.

### MASVS-CODE: Qualidade de Código e Mitigação de Exploit

Problemas tradicionais de injeção e gerenciamento de memória não são frequentemente vistos em aplicativos móveis devido à menor superfície de ataque. Aplicativos móveis interagem principalmente com o serviço backend confiável e a UI, então mesmo que muitas vulnerabilidades de estouro de buffer existam no aplicativo, essas vulnerabilidades geralmente não abrem vetores de ataque úteis. O mesmo se aplica a exploits de navegador, como cross-site scripting (XSS permite que atacantes injetem scripts em páginas web) que são muito prevalentes em aplicativos web. No entanto, sempre há exceções. XSS é teoricamente possível em mobile em alguns casos, mas é muito raro ver problemas de XSS que um indivíduo possa explorar.

Esta proteção contra problemas de injeção e gerenciamento de memória não significa que desenvolvedores de aplicativos podem se safar escrevendo código descuidado. Seguir as melhores práticas de segurança resulta em builds de release endurecidos (seguros) que são resilientes contra adulteração. Recursos de segurança gratuitos oferecidos por compiladores e SDKs móveis ajudam a aumentar a segurança e mitigar ataques.

### MASVS-RESILIENCE: Anti-Adulteração e Anti-Engenharia Reversa

Há três coisas que você nunca deve trazer à tona em conversas educadas: religião, política e ofuscação de código. Muitos especialistas em segurança descartam proteções do lado do cliente completamente. No entanto, controles de proteção de software são amplamente usados no mundo de aplicativos móveis, então testadores de segurança precisam de maneiras de lidar com essas proteções. Acreditamos que há benefício em proteções do lado do cliente se forem empregadas com um propósito claro e expectativas realistas em mente e não forem usadas para substituir controles de segurança.