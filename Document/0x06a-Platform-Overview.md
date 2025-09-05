
# Visão Geral da Plataforma iOS

iOS é um sistema operacional móvel que alimenta dispositivos móveis Apple, incluindo iPhone, iPad e iPod Touch. Também é a base para o Apple tvOS, que herda muitas funcionalidades do iOS. Esta seção introduz a plataforma iOS do ponto de vista da arquitetura. As seguintes cinco áreas-chave são discutidas:

1. Arquitetura de segurança iOS
2. Estrutura de aplicativos iOS
3. Comunicação Inter-Processos (IPC)
4. Publicação de aplicativos iOS
5. Superfície de Ataque de Aplicativos iOS

Como o sistema operacional desktop da Apple macOS (anteriormente OS X), o iOS é baseado no Darwin, um sistema operacional Unix de código aberto desenvolvido pela Apple. O kernel do Darwin é o XNU ("X is Not Unix"), um kernel híbrido que combina componentes dos kernels Mach e FreeBSD.

No entanto, os aplicativos iOS são executados em um ambiente mais restrito do que suas contrapartes desktop. Os aplicativos iOS são isolados uns dos outros no nível do sistema de arquivos e são significativamente limitados em termos de acesso à API do sistema.

Para proteger os usuários de aplicativos maliciosos, a Apple restringe e controla o acesso aos aplicativos que têm permissão para serem executados em dispositivos iOS. A App Store da Apple é a única plataforma oficial de distribuição de aplicativos. Lá os desenvolvedores podem oferecer seus aplicativos e os consumidores podem comprar, baixar e instalar aplicativos. Este estilo de distribuição difere do Android, que suporta várias lojas de aplicativos e sideloading (instalar um aplicativo em seu dispositivo iOS sem usar a App Store oficial). No iOS, sideloading normalmente se refere ao método de instalação de aplicativos via USB, embora existam outros métodos de distribuição de aplicativos iOS empresariais que não usam a App Store sob o [Apple Developer Enterprise Program](https://developer.apple.com/programs/enterprise/ "Apple Developer Enterprise Program").

No passado, o sideloading era possível apenas com jailbreak ou soluções alternativas complicadas. Com iOS 9 ou superior, é possível fazer [sideloading via Xcode](https://forums.developer.apple.com/forums/thread/91370).

Os aplicativos iOS são isolados uns dos outros via sandbox iOS da Apple (historicamente chamado de Seatbelt), um mecanismo de controle de acesso obrigatório (MAC) que descreve os recursos que um aplicativo pode e não pode acessar. Comparado às extensas facilidades IPC Binder do Android, o iOS oferece muito poucas opções de IPC (Comunicação Inter-Processos), minimizando a superfície de ataque potencial.

Hardware uniforme e integração apertada de hardware/software criam outra vantagem de segurança. Cada dispositivo iOS oferece recursos de segurança, como inicialização segura, Keychain com suporte de hardware e criptografia de sistema de arquivos (referida como proteção de dados no iOS). As atualizações do iOS geralmente são rapidamente distribuídas para uma grande porcentagem de usuários, diminuindo a necessidade de suportar versões mais antigas e desprotegidas do iOS.

Apesar das inúmeras forças do iOS, os desenvolvedores de aplicativos iOS ainda precisam se preocupar com segurança. Proteção de dados, Keychain, autenticação Touch ID/Face ID e segurança de rede ainda deixam uma grande margem para erros. Nos capítulos seguintes, descrevemos a arquitetura de segurança iOS, explicamos uma metodologia básica de teste de segurança e fornecemos como-fazer de engenharia reversa.

## Arquitetura de Segurança iOS

A [arquitetura de segurança iOS](https://www.apple.com/business/docs/iOS_Security_Guide.pdf "Apple iOS Security Guide"), documentada oficialmente pela Apple no iOS Security Guide, consiste em seis recursos principais. Este guia de segurança é atualizado pela Apple para cada versão principal do iOS:

- Segurança de Hardware
- Inicialização Segura
- Code Signing
- Sandbox
- Criptografia e Proteção de Dados
- Mitigações Gerais de Exploit

<img src="Images/Chapters/0x06a/iOS_Security_Architecture.png" width="200px" />

### Segurança de Hardware

A arquitetura de segurança iOS faz bom uso de recursos de segurança baseados em hardware que melhoram o desempenho geral. Cada dispositivo iOS vem com duas chaves AES (Advanced Encryption Standard) de 256 bits incorporadas. Os IDs únicos do dispositivo (UIDs) e IDs de grupo de dispositivos (GIDs) são chaves AES de 256 bits fundidas (UID) ou compiladas (GID) no Application Processor (AP) e Secure Enclave Processor (SEP) durante a fabricação. Não há maneira direta de ler essas chaves com software ou interfaces de depuração como JTAG. As operações de criptografia e descriptografia são realizadas por motores criptográficos AES de hardware que têm acesso exclusivo a essas chaves.

O GID é um valor compartilhado por todos os processadores em uma classe de dispositivos usado para evitar adulteração de arquivos de firmware e outras tarefas criptográficas não diretamente relacionadas aos dados privados do usuário. UIDs, que são exclusivos para cada dispositivo, são usados para proteger a hierarquia de chaves usada para criptografia de sistema de arquivos no nível do dispositivo. Como os UIDs não são registrados durante a fabricação, nem mesmo a Apple pode restaurar as chaves de criptografia de arquivo para um dispositivo específico.

Para permitir a exclusão segura de dados sensíveis na memória flash, os dispositivos iOS incluem um recurso chamado [Effaceable Storage](https://www.apple.com/business/docs/iOS_Security_Guide.pdf "iOS Security Guide"). Este recurso fornece acesso direto de baixo nível à tecnologia de armazenamento, tornando possível apagar com segurança blocos selecionados.

### Inicialização Segura

Quando um dispositivo iOS é ligado, ele lê as instruções iniciais da memória somente leitura conhecida como Boot ROM, que inicializa o sistema. O Boot ROM contém código imutável e a Apple Root CA, que é gravada no chip de silício durante o processo de fabricação, criando assim a raiz de confiança. Em seguida, o Boot ROM verifica se a assinatura do LLB (Low Level Bootloader) está correta, e o LLB verifica se a assinatura do bootloader iBoot também está correta. Após a validação da assinatura, o iBoot verifica a assinatura do próximo estágio de inicialização, que é o kernel iOS. Se qualquer uma dessas etapas falhar, o processo de inicialização será encerrado imediatamente e o dispositivo entrará no modo de recuperação e exibirá a [tela de restauração](https://support.apple.com/en-us/HT203122 "If you see the Restore screen on your iPhone, iPad, or iPod touch"). No entanto, se o Boot ROM falhar ao carregar, o dispositivo entrará em um modo especial de recuperação de baixo nível chamado Device Firmware Upgrade (DFU). Este é o último recurso para restaurar o dispositivo ao seu estado original. Neste modo, o dispositivo não mostrará nenhum sinal de atividade; ou seja, sua tela não exibirá nada.

Este processo inteiro é chamado de "Cadeia de Inicialização Segura". Seu propósito é focado em verificar a integridade do processo de inicialização, garantindo que o sistema e seus componentes sejam escritos e distribuídos pela Apple. A cadeia de inicialização segura consiste no kernel, no bootloader, na extensão do kernel e no firmware da baseband.

### Code Signing

A Apple implementou um elaborado sistema DRM para garantir que apenas código aprovado pela Apple seja executado em seus dispositivos, ou seja, código assinado pela Apple. Em outras palavras, você não será capaz de executar qualquer código em um dispositivo iOS que não tenha sido jailbroken, a menos que a Apple explicitamente permita. Os usuários finais devem instalar aplicativos apenas através da App Store oficial da Apple. Por esta razão (e outras), o iOS foi [comparado a uma prisão de cristal](https://www.eff.org/deeplinks/2012/05/apples-crystal-prison-and-future-of-open-platforms "Apple\'s Crystal Prison and the Future of Open Platforms").

Um perfil de desenvolvedor e um certificado assinado pela Apple são necessários para implantar e executar um aplicativo.
Os desenvolvedores precisam se registrar na Apple, ingressar no [Apple Developer Program](https://developer.apple.com/support/compare-memberships/ "Membership for Apple Developer Program") e pagar uma assinatura anual para obter toda a gama de possibilidades de desenvolvimento e implantação. Há também uma conta de desenvolvedor gratuita que permite compilar e implantar aplicativos (mas não distribuí-los na App Store) via sideloading.

<img src="Images/Chapters/0x06a/code_signing.png" width="400px" />

De acordo com a [Documentação de Desenvolvedor da Apple Arquivada](https://developer.apple.com/library/archive/documentation/Security/Conceptual/CodeSigningGuide/AboutCS/AboutCS.html#//apple_ref/doc/uid/TP40005929-CH3-SW3), a assinatura de código consiste em três partes:

- Um selo. Esta é uma coleção de checksums ou hashes das várias partes do código, criada pelo software de assinatura de código. O selo pode ser usado no momento da verificação para detectar alterações.
- Uma assinatura digital. O software de assinatura de código criptografa o selo usando a identidade do signatário para criar uma assinatura digital. Isso garante a integridade do selo.
- Requisitos de código. Estas são as regras que regem a verificação da assinatura de código. Dependendo dos objetivos, alguns são inerentes ao verificador, enquanto outros são especificados pelo signatário e selados com o resto do código.

Saiba mais:

- [Guia de Assinatura de Código (Documentação de Desenvolvedor da Apple Arquivada)](https://developer.apple.com/library/archive/documentation/Security/Conceptual/CodeSigningGuide/Introduction/Introduction.html)
- [Assinatura de Código (Documentação do Desenvolvedor Apple)](https://developer.apple.com/support/code-signing/)
- [Desmistificando a Assinatura de Código iOS](https://medium.com/csit-tech-blog/demystifying-ios-code-signature-309d52c2ff1d)

### Criptografia e Proteção de Dados

_FairPlay Code Encryption_ é aplicado a aplicativos baixados da App Store. FairPlay foi desenvolvido como um DRM ao comprar conteúdo multimídia. Originalmente, a criptografia FairPlay era aplicada a streams MPEG e QuickTime, mas os mesmos conceitos básicos também podem ser aplicados a arquivos executáveis. A ideia básica é a seguinte: Uma vez que você registra uma nova conta de usuário Apple, ou Apple ID, um par de chaves pública/privada será criado e atribuído à sua conta. A chave privada é armazenada com segurança em seu dispositivo. Isso significa que o código criptografado FairPlay só pode ser descriptografado em dispositivos associados à sua conta. A reversão da criptografia FairPlay é geralmente obtida executando o aplicativo no dispositivo e então despejando o código descriptografado da memória (veja também "Teste de Segurança Básico no iOS").

A Apple construiu criptografia no hardware e firmware de seus dispositivos iOS desde o lançamento do iPhone 3GS. Cada dispositivo tem um motor criptográfico baseado em hardware dedicado que fornece uma implementação da criptografia AES de 256 bits e dos algoritmos de hash SHA-1. Além disso, há um identificador único (UID) incorporado no hardware de cada dispositivo com uma chave AES de 256 bits fundida no Application Processor. Este UID é único e não registrado em outro lugar. No momento da escrita, nem o software nem o firmware podem ler diretamente o UID. Como a chave é queimada no chip de silício, ela não pode ser adulterada ou contornada. Apenas o motor criptográfico pode acessá-la.

Construir criptografia na arquitetura física a torna um recurso de segurança padrão que pode criptografar todos os dados armazenados em um dispositivo iOS. Como resultado, a proteção de dados é implementada no nível do software e funciona com a criptografia de hardware e firmware para fornecer mais segurança.

Quando a proteção de dados está habilitada, simplesmente estabelecendo um código de acesso no dispositivo móvel, cada arquivo de dados é associado a uma classe de proteção específica. Cada classe suporta um nível diferente de acessibilidade e protege os dados com base em quando os dados precisam ser acessados. As operações de criptografia e descriptografia associadas a cada classe são baseadas em múltiplos mecanismos de chave que utilizam o UID do dispositivo e código de acesso, uma chave de classe, uma chave de sistema de arquivos e uma chave por arquivo. A chave por arquivo é usada para criptografar o conteúdo do arquivo. A chave de classe é encapsulada em torno da chave por arquivo e armazenada nos metadados do arquivo. A chave do sistema de arquivos é usada para criptografar os metadados. O UID e o código de acesso protegem a chave de classe. Esta operação é invisível para os usuários. Para habilitar a proteção de dados, o código de acesso deve ser usado ao acessar o dispositivo. O código de acesso desbloqueia o dispositivo. Combinado com o UID, o código de acesso também cria chaves de criptografia iOS que são mais resistentes a hacking e ataques de força bruta. Habilitar a proteção de dados é a principal razão para os usuários usarem códigos de acesso em seus dispositivos.

### Sandbox

O [appsandbox](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html "File System Basics") é uma tecnologia de controle de acesso iOS. É aplicada no nível do kernel. Seu propósito é limitar danos ao sistema e dados do usuário que podem ocorrer quando um aplicativo é comprometido.

O sandboxing tem sido um recurso de segurança central desde o primeiro lançamento do iOS. Todos os aplicativos de terceiros são executados sob o mesmo usuário (`mobile`), e apenas alguns aplicativos e serviços do sistema são executados como `root` (ou outros usuários específicos do sistema). Aplicativos iOS regulares são confinados a um _container_ que restringe o acesso aos próprios arquivos do aplicativo e a um número muito limitado de APIs do sistema. O acesso a todos os recursos (como arquivos, soquetes de rede, IPCs e memória compartilhada) é controlado pelo sandbox. Essas restrições funcionam da seguinte forma [#levin]:

- O processo do aplicativo é restrito ao seu próprio diretório (sob /var/mobile/Containers/ Bundle/Application/ ou /var/containers/Bundle/Application/, dependendo da versão do iOS) via um processo semelhante a chroot.
- As chamadas de sistema `mmap` e `mmprotect` são modificadas para impedir que os aplicativos tornem páginas de memória graváveis executáveis e parem processos de executar código gerado dinamicamente. Em combinação com code signing e FairPlay, isso limita estritamente qual código pode ser executado sob circunstâncias específicas (por exemplo, todo o código em aplicativos distribuídos via App Store é aprovado pela Apple).
- Os processos são isolados uns dos outros, mesmo que sejam de propriedade do mesmo UID no nível do sistema operacional.
- Drivers de hardware não podem ser acessados diretamente. Em vez disso, eles devem ser acessados através dos frameworks públicos da Apple.

### Mitigações Gerais de Exploit

O iOS implementa randomização do layout do espaço de endereço (ASLR) e bit eXecute Never (XN) para mitigar ataques de execução de código.

ASLR randomiza a localização de memória do arquivo executável do programa, dados, heap e stack toda vez que o programa é executado. Como as bibliotecas compartilhadas devem ser estáticas para serem acessadas por múltiplos processos, os endereços das bibliotecas compartilhadas são randomizados toda vez que o OS é inicializado em vez de toda vez que o programa é invocado. Isso torna endereços de memória de funções e bibliotecas específicas difíceis de prever, impedindo assim ataques como o ataque return-to-libc, que envolve os endereços de memória de funções básicas do libc.

O mecanismo XN permite que o iOS marque segmentos de memória selecionados de um processo como não executáveis. No iOS, a stack e heap do processo de processos em modo de usuário são marcadas como não executáveis. Páginas que são graváveis não podem ser marcadas como executáveis ao mesmo tempo. Isso impede que invasores executem código de má
injetado na stack ou heap.

## Desenvolvimento de Software no iOS

Como outras plataformas, a Apple fornece um Software Development Kit (SDK) que ajuda desenvolvedores a desenvolver, instalar, executar e testar aplicativos iOS nativos. Xcode é um Ambiente de Desenvolvimento Integrado (IDE) para desenvolvimento de software Apple. Aplicativos iOS são desenvolvidos em Objective-C ou Swift.

Objective-C é uma linguagem de programação orientada a objetos que adiciona mensagens no estilo Smalltalk à linguagem de programação C. É usado no macOS para desenvolver aplicativos desktop e no iOS para desenvolver aplicativos móveis. Swift é o sucessor do Objective-C e permite interoperabilidade com Objective-C.

Swift foi introduzido com Xcode 6 em 2014.

Em um dispositivo não jailbroken, existem duas maneiras de instalar um aplicativo fora da App Store:

1. via Enterprise Mobile Device Management. Isso requer um certificado corporativo assinado pela Apple.
2. via sideloading, ou seja, assinando um aplicativo com um certificado de desenvolvedor e instalando-o no dispositivo via Xcode (ou Cydia Impactor). Um número limitado de dispositivos pode ser instalado com o mesmo certificado.

## Aplicativos no iOS

Aplicativos iOS são distribuídos em arquivos IPA (iOS App Store Package). O arquivo IPA é um arquivo compactado ZIP que contém todo o código e recursos necessários para executar o aplicativo.

Arquivos IPA têm uma estrutura de diretório incorporada. O exemplo abaixo mostra esta estrutura em alto nível:

- A pasta `/Payload/` contém todos os dados do aplicativo. Voltaremos ao conteúdo desta pasta com mais detalhes.
- `/Payload/Application.app` contém os dados do aplicativo em si (código compilado ARM) e recursos estáticos associados.
- `/iTunesArtwork` é uma imagem PNG de 512x512 pixels usada como ícone do aplicativo.
- `/iTunesMetadata.plist` contém várias informações, incluindo nome e ID do desenvolvedor, identificador do bundle, informações de copyright, gênero, nome do aplicativo, data de lançamento, data de compra, etc.
- `/WatchKitSupport/WK` é um exemplo de um bundle de extensão. Este bundle específico contém o delegado de extensão e os controladores para gerenciar as interfaces e responder às interações do usuário em um Apple Watch.

### Cargas IPA - Uma Visão Mais Detalhada

Vamos dar uma olhada mais de perto nos diferentes arquivos no container IPA. A Apple usa uma estrutura relativamente plana com poucos diretórios extras para economizar espaço em disco e simplificar o acesso a arquivos. O diretório de bundle de nível superior contém o arquivo executável do aplicativo e todos os recursos que o aplicativo usa (por exemplo, o ícone do aplicativo, outras imagens e conteúdo localizado.

- **MyApp**: O arquivo executável contendo o código fonte compilado (ilegível) do aplicativo.
- **Application**: Ícones do aplicativo.
- **Info.plist**: Informações de configuração, como ID do bundle, número da versão e nome de exibição do aplicativo.
- **Launch images**: Imagens mostrando a interface inicial do aplicativo em uma orientação específica. O sistema usa uma das imagens de inicialização fornecidas como plano de fundo temporário até que o aplicativo seja totalmente carregado.
- **MainWindow.nib**: Objetos de interface padrão que são carregados quando o aplicativo é iniciado. Outros objetos de interface são então carregados de outros arquivos nib ou criados programaticamente pelo aplicativo.
- **Settings.bundle**: Preferências específicas do aplicativo a serem exibidas no aplicativo Configurações.
- **Custom resource files**: Recursos não localizados são colocados no diretório de nível superior e recursos localizados são colocados em subdiretórios específicos de idioma do bundle do aplicativo. Os recursos incluem arquivos nib, imagens, arquivos de som, arquivos de configuração, arquivos de strings e quaisquer outros arquivos de dados personalizados que o aplicativo usa.

Uma pasta language.lproj existe para cada idioma que o aplicativo suporta. Ela contém um storyboard e um arquivo de strings.

- Um storyboard é uma representação visual da interface do usuário do aplicativo iOS. Ele mostra telas e as conexões entre essas telas.
- O formato do arquivo de strings consiste em um ou mais pares chave-valor e comentários opcionais.

<img src="Images/Chapters/0x06a/iOS_project_folder.png" width="400px" />

Em um dispositivo jailbroken, você pode recuperar o IPA para um aplicativo iOS instalado usando diferentes ferramentas que permitem descriptografar o binário principal do aplicativo e reconstruir o arquivo IPA. Da mesma forma, em um dispositivo jailbroken você pode instalar o arquivo IPA com @MASTG-TOOL-0138. Durante avaliações de segurança móvel, os desenvolvedores geralmente fornecem o IPA diretamente. Eles podem enviar o arquivo real ou fornecer acesso à plataforma de distribuição específica de desenvolvimento que usam, por exemplo [TestFlight](https://developer.apple.com/testflight/ "TestFlight") ou [Visual Studio App Center](https://appcenter.ms/ "Visual Studio App Center").

### Permissões de Aplicativo

Em contraste com aplicativos Android (antes do Android 6.0 (API level 23)), aplicativos iOS não têm permissões pré-atribuídas. Em vez disso, o usuário é solicitado a conceder permissão durante o tempo de execução, quando o aplicativo tenta usar uma API sensível pela primeira vez. Aplicativos que tiveram permissões concedidas são listados no menu Configurações > Privacidade, permitindo que o usuário modifique a configuração específica do aplicativo. A Apple chama este conceito de permissão de [controles de privacidade](https://support.apple.com/en-sg/HT203033 "Apple - About privacy and Location Services in iOS 8 and later").

Desenvolvedores iOS não podem definir permissões solicitadas diretamente, estas serão solicitadas indiretamente ao acessar APIs sensíveis. Por exemplo, ao acessar os contatos de um usuário, qualquer chamada para CNContactStore bloqueia o aplicativo enquanto o usuário é solicitado a conceder ou negar acesso. A partir do iOS 10.0, os aplicativos devem incluir chaves de descrição de uso para os tipos de permissões que solicitam e dados que precisam acessar (por exemplo, NSContactsUsageDescription).

As seguintes APIs [requerem permissão do usuário](https://www.apple.com/business/docs/iOS_Security_Guide.pdf "iOS Security Guide. Page 62"):

- Contatos
- Microfone
- Calendários
- Câmera
- Lembretes
- HomeKit
- Fotos
- Saúde
- Atividade de movimento e fitness
- Reconhecimento de fala
- Serviços de Localização
- Compartilhamento Bluetooth
- Biblioteca de Mídia
- Contas de mídia social

### DeviceCheck

O framework DeviceCheck, incluindo seus componentes DeviceCheck e App Attest, ajuda a prevenir uso fraudulento de seus serviços. Ele consiste em um framework que você usa do seu aplicativo e um servidor Apple que é acessível apenas ao seu próprio servidor. DeviceCheck permite armazenar informações persistentemente no dispositivo e nos servidores Apple. As informações armazenadas permanecem intactas através da reinstalação do aplicativo, transferências de dispositivo ou reset, com a opção de resetar esses dados periodicamente.

DeviceCheck é tipicamente usado para mitigar fraudes restringindo o acesso a recursos sensíveis. Por exemplo, limitando promoções a uma vez por dispositivo, identificando e sinalizando dispositivos fraudulentos, etc. No entanto, definitivamente não pode prevenir todas as fraudes. Por exemplo, ele [não se destina a detectar sistemas operacionais comprometidos](https://swiftrocks.com/app-attest-apple-protect-ios-jailbreak "App Attest: How to prevent an iOS app's APIs from being abused") (também conhecido como detecção de jailbreak).

Para mais informações, consulte a [documentação do DeviceCheck](https://developer.apple.com/documentation/devicecheck "DeviceCheck documentation").

#### App Attest

App Attest, disponível sob o framework DeviceCheck, ajuda a verificar instâncias do aplicativo em execução em um dispositivo, permitindo que aplicativos anexem uma asserção com suporte de hardware a solicitações, garantindo que elas se originem do aplicativo legítimo em um dispositivo Apple genuíno. Este recurso ajuda a impedir que aplicativos modificados se comuniquem com seu servidor.

O processo envolve gerar e validar chaves criptográficas, junto com um conjunto de verificações realizadas pelo seu servidor, garantindo a autenticidade da solicitação. É importante notar que, embora o App Attest melhore a segurança, ele não garante proteção completa contra todas as formas de atividades fraudulentas.

Para informações mais detalhadas, consulte a sessão [WWDC 2021](https://developer.apple.com/videos/play/wwdc2021/10244 "WWDC 2021"), juntamente com a ["documentação do DeviceCheck"](https://developer.apple.com/documentation/devicecheck/) e ["Validando aplicativos que se conectam ao seu servidor"](https://developer.apple.com/documentation/devicecheck/validating-apps-that-connect-to-your-server).