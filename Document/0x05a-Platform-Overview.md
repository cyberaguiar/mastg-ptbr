
# Visão Geral da Plataforma Android

Este capítulo apresenta a plataforma Android do ponto de vista da arquitetura. As cinco áreas principais a seguir são discutidas:

1. Arquitetura Android
2. Segurança Android: abordagem de defesa em profundidade
3. Estrutura de aplicativos Android
4. Publicação de aplicativos Android
5. Superfície de ataque de aplicativos Android

Visite o site oficial da [documentação do desenvolvedor Android](https://developer.android.com/index.html "Guia do Desenvolvedor Android") para mais detalhes sobre a plataforma Android.

## Arquitetura Android

[Android](https://en.wikipedia.org/wiki/Android_(operating_system) "Android (Sistema Operacional)") é uma plataforma open source baseada em Linux desenvolvida pela [Open Handset Alliance](https://www.openhandsetalliance.com/) (um consórcio liderado pelo Google), que serve como um sistema operacional (SO) móvel. Hoje a plataforma é a base para uma ampla variedade de tecnologias modernas, como telefones móveis, tablets, tecnologia vestível, TVs e outros dispositivos inteligentes. Builds típicas do Android são enviadas com uma variedade de aplicativos pré-instalados ("stock") e suportam a instalação de aplicativos de terceiros através da Google Play store e outros marketplaces.

A pilha de software do Android é composta por várias camadas diferentes. Cada camada define interfaces e oferece serviços específicos.

<img src="Images/Chapters/0x05a/android_software_stack.png" width="400px" />

**Kernel:** No nível mais baixo, o Android é baseado em uma [variação do Kernel Linux](https://source.android.com/devices/architecture/kernel) contendo algumas adições significativas, incluindo [Low Memory Killer](https://source.android.com/devices/tech/perf/lmkd), wake locks, o driver [Binder IPC](https://source.android.com/devices/architecture/hidl/binder-ipc), etc. Para o propósito do MASTG, focaremos na parte do SO em modo de usuário, onde o Android difere significativamente de uma distribuição Linux típica. Os dois componentes mais importantes para nós são o runtime gerenciado usado por aplicativos (ART/Dalvik) e [Bionic](https://en.wikipedia.org/wiki/Bionic_(software) "Android (Bionic)"), a versão do Android da glibc, a biblioteca GNU C.

**HAL:** No topo do kernel, a Camada de Abstração de Hardware (HAL) define uma interface padrão para interagir com componentes de hardware integrados. Várias implementações HAL são empacotadas em módulos de biblioteca compartilhada que o sistema Android chama quando necessário. Esta é a base para permitir que aplicativos interajam com o hardware do dispositivo. Por exemplo, permite que um aplicativo de telefone stock use o microfone e alto-falante do dispositivo.

**Ambiente de Runtime:** Aplicativos Android são escritos em Java e Kotlin e então compilados para [bytecode Dalvik](https://source.android.com/devices/tech/dalvik/dalvik-bytecode) que pode então ser executado usando um runtime que interpreta as instruções do bytecode e as executa no dispositivo de destino. Para o Android, este é o [Android Runtime (ART)](https://source.android.com/devices/tech/dalvik/configure#how_art_works). Isto é semelhante à [JVM (Java Virtual Machine)](https://en.wikipedia.org/wiki/Java_virtual_machine) para aplicativos Java, ou o Mono Runtime para aplicativos .NET.

O bytecode Dalvik é uma versão otimizada do bytecode Java. Ele é criado primeiro compilando o código Java ou Kotlin para bytecode Java, usando os compiladores javac e kotlinc respectivamente, produzindo arquivos .class. Finalmente, o bytecode Java é convertido para bytecode Dalvik usando a ferramenta d8. O bytecode Dalvik é empacotado dentro de arquivos APK e AAB na forma de arquivos .dex e é usado por um runtime gerenciado no Android para executá-lo no dispositivo.

<img src="Images/Chapters/0x05a/java_vs_dalvik.png" width="400px" />

Antes do Android 5.0 (API level 21), o Android executava bytecode na Máquina Virtual Dalvik (DVM), onde era traduzido para código de máquina no momento da execução, um processo conhecido como compilação _just-in-time_ (JIT). Isso permite que o runtime se beneficie da velocidade do código compilado enquanto mantém a flexibilidade da interpretação de código.

Desde o Android 5.0 (API level 21), o Android executa bytecode no Android Runtime (ART) que é o sucessor do DVM. O ART fornece desempenho melhorado, bem como informações de contexto em relatórios de crash nativo de aplicativos, incluindo informações de stack Java e nativo. Ele usa a mesma entrada de bytecode Dalvik para manter compatibilidade com versões anteriores. No entanto, o ART executa o bytecode Dalvik de forma diferente, usando uma combinação híbrida de compilação _ahead-of-time_ (AOT), _just-in-time_ (JIT) e guiada por perfil.

- **AOT** pré-compila bytecode Dalvik em código nativo, e o código gerado será salvo em disco com a extensão .oat (binário ELF). A ferramenta dex2oat pode ser usada para realizar a compilação e pode ser encontrada em /system/bin/dex2oat em dispositivos Android. A compilação AOT é executada durante a instalação do aplicativo. Isso faz com que o aplicativo inicie mais rápido, pois nenhuma compilação é mais necessária. No entanto, isso também significa que o tempo de instalação aumenta em comparação com a compilação JIT. Além disso, como os aplicativos são sempre otimizados contra a versão atual do SO, isso significa que as atualizações de software recompilarão todos os aplicativos previamente compilados, resultando em um aumento significativo no tempo de atualização do sistema. Finalmente, a compilação AOT compilará o aplicativo inteiro, mesmo que certas partes nunca sejam usadas pelo usuário.
- **JIT** acontece em tempo de execução.
- **Compilação guiada por perfil** é uma abordagem híbrida que foi introduzida no Android 7 (API level 24) para combater as desvantagens do AOT. Primeiro, o aplicativo usará compilação JIT, e o Android mantém o controle de todas as partes do aplicativo que são frequentemente usadas. Esta informação é armazenada em um perfil de aplicativo e quando o dispositivo está ocioso, um daemon de compilação (dex2oat) é executado que compila AOT os caminhos de código frequentes identificados a partir do perfil.

<img src="Images/Chapters/0x05a/java2oat.png" width="100%" />

Fonte: <https://lief-project.github.io/doc/latest/tutorials/10_android_formats.html>

**Sandboxing:** Aplicativos Android não têm acesso direto a recursos de hardware, e cada aplicativo é executado em sua própria máquina virtual ou sandbox. Isso permite que o SO tenha controle preciso sobre recursos e acesso à memória no dispositivo. Por exemplo, um aplicativo em crash não afeta outros aplicativos em execução no mesmo dispositivo. O Android controla o número máximo de recursos do sistema alocados para aplicativos, impedindo que qualquer aplicativo monopolize muitos recursos. Ao mesmo tempo, este design de sandbox pode ser considerado como um dos muitos princípios na estratégia global de defesa em profundidade do Android. Um aplicativo de terceiros malicioso, com baixos privilégios, não deve ser capaz de escapar de seu próprio runtime e ler a memória de um aplicativo vítima no mesmo dispositivo. Na seção seguinte, damos uma olhada mais de perto nas diferentes camadas de defesa no sistema operacional Android. Saiba mais na seção ["Isolamento de Software"](#isolamento-de-software).

Você pode encontrar informações mais detalhadas no artigo do Google Source ["Android Runtime (ART)"](https://source.android.com/devices/tech/dalvik/configure#how_art_works), no [livro "Android Internals" de Jonathan Levin](http://newandroidbook.com/) e no [post do blog "Android 101" de @_qaz_qaz](https://secrary.com/android-reversing/android101/).

## Segurança Android: Abordagem de Defesa em Profundidade

A arquitetura Android implementa diferentes camadas de segurança que, juntas, permitem uma abordagem de defesa em profundidade. Isso significa que a confidencialidade, integridade ou disponibilidade de dados sensíveis do usuário ou aplicativos não depende de uma única medida de segurança. Esta seção traz uma visão geral das diferentes camadas de defesa que o sistema Android fornece. A estratégia de segurança pode ser grosseiramente categorizada em quatro domínios distintos, cada um focando em proteger contra certos modelos de ataque.

- Segurança em todo o sistema
- Isolamento de software
- Segurança de rede
- Anti-exploração

### Segurança em todo o sistema

#### Criptografia de dispositivo

O Android suporta criptografia de dispositivo desde o Android 2.3.4 (API level 10) e passou por algumas grandes mudanças desde então. O Google impôs que todos os dispositivos executando Android 6.0 (API level 23) ou superior deveriam suportar criptografia de armazenamento, embora alguns dispositivos de baixo custo tenham sido isentos porque impactaria significativamente seu desempenho.

- [Criptografia de Disco Completo (FDE)](https://source.android.com/security/encryption/full-disk "Criptografia de Disco Completo"): Android 5.0 (API level 21) e superior suportam criptografia de disco completo. Esta criptografia usa uma única chave protegida pela senha do dispositivo do usuário para criptografar e descriptografar a partição de dados do usuário. Este tipo de criptografia é agora considerado obsoleto e a criptografia baseada em arquivo deve ser usada sempre que possível. A criptografia de disco completo tem desvantagens, como não poder receber chamadas ou não ter alarmes operacionais após uma reinicialização se o usuário não inserir a senha para desbloquear.

- [Criptografia Baseada em Arquivo (FBE)](https://source.android.com/security/encryption/file-based "Criptografia Baseada em Arquivo"): Android 7.0 (API level 24) suporta criptografia baseada em arquivo. A criptografia baseada em arquivo permite que diferentes arquivos sejam criptografados com chaves diferentes para que possam ser decifrados independentemente. Dispositivos que suportam este tipo de criptografia também suportam Direct Boot. O Direct Boot permite que o dispositivo tenha acesso a recursos como alarmes ou serviços de acessibilidade mesmo se o usuário não tiver desbloqueado o dispositivo.

> Nota: você pode ouvir falar de [Adiantum](https://github.com/google/adiantum "Adiantum"), que é um método de criptografia projetado para dispositivos executando Android 9 (API level 28) e superior cujas CPUs carecem de instruções AES. **Adiantum é relevante apenas para desenvolvedores de ROM ou fornecedores de dispositivos**, o Android não fornece uma API para desenvolvedores usarem Adiantum a partir de aplicativos. Como recomendado pelo Google, Adiantum não deve ser usado ao enviar dispositivos baseados em ARM com ARMv8 Cryptography Extensions ou dispositivos baseados em x86 com AES-NI. AES é mais rápido nessas plataformas.
>
> Mais informações estão disponíveis na [documentação do Android](https://source.android.com/security/encryption/adiantum "Adiantum").

#### Ambiente de Execução Confiável (TEE)

Para que o sistema Android execute criptografia, ele precisa de uma maneira de gerar, importar e armazenar chaves criptográficas com segurança. Estamos essencialmente deslocando o problema de manter dados sensíveis seguros para manter uma chave criptográfica segura. Se o invasor puder despejar ou adivinhar a chave criptográfica, os dados sensíveis criptografados podem ser recuperados.

O Android oferece um ambiente de execução confiável em hardware dedicado para resolver o problema de gerar e proteger chaves criptográficas com segurança. Isso significa que um componente de hardware dedicado no sistema Android é responsável por lidar com material de chave criptográfica. Três módulos principais são responsáveis por isso:

- [KeyStore com suporte de hardware](https://source.android.com/security/keystore): Este módulo oferece serviços criptográficos para o SO Android e aplicativos de terceiros. Ele permite que aplicativos realizem operações criptográficas sensíveis em um TEE sem expor o material da chave criptográfica.

- [StrongBox](https://developer.android.com/training/articles/keystore#HardwareSecurityModule): No Android 9 (Pie), o StrongBox foi introduzido, outra abordagem para implementar um KeyStore com suporte de hardware. Enquanto antes do Android 9 Pie, um KeyStore com suporte de hardware seria qualquer implementação TEE que fica fora do kernel do SO Android. O StrongBox é um chip de hardware separado completo que é adicionado ao dispositivo no qual o KeyStore é implementado e é claramente definido na documentação do Android. Você pode verificar programaticamente se uma chave reside no StrongBox e se reside, você pode ter certeza de que ela é protegida por um módulo de segurança de hardware que tem sua própria CPU, armazenamento seguro e Gerador de Números Aleatórios Verdadeiro (TRNG). Todas as operações criptográficas sensíveis acontecem neste chip, nos limites seguros do StrongBox.

- [GateKeeper](https://source.android.com/security/authentication/gatekeeper): O módulo GateKeeper permite autenticação de padrão e senha do dispositivo. As operações sensíveis de segurança durante o processo de autenticação acontecem dentro do TEE que está disponível no dispositivo. O GateKeeper consiste em três componentes principais, (1) `gatekeeperd` que é o serviço que expõe o GateKeeper, (2) GateKeeper HAL, que é a interface de hardware e (3) a implementação TEE que é o software real que implementa a funcionalidade GateKeeper no TEE.

#### Inicialização Verificada

Precisamos ter uma maneira de garantir que o código que está sendo executado em dispositivos Android venha de uma fonte confiável e que sua integridade não esteja comprometida. Para alcançar isso, o Android introduziu o conceito de inicialização verificada. O objetivo da inicialização verificada é estabelecer uma relação de confiança entre o hardware e o código real que executa neste hardware. Durante a sequência de inicialização verificada, uma cadeia completa de confiança é estabelecida começando da Raiz de Confiança (RoT) protegida por hardware até o sistema final que está sendo executado, passando por e verificando todas as fases de inicialização necessárias. Quando o sistema Android é finalmente inicializado, você pode ter certeza de que o sistema não foi adulterado. Você tem prova criptográfica de que o código que está sendo executado é aquele pretendido pelo OEM e não um que foi alterado maliciosamente ou acidentalmente.

Mais informações estão disponíveis na [documentação do Android](https://source.android.com/security/verifiedboot).

### Isolamento de Software

#### Usuários e Grupos Android

Embora o sistema operacional Android seja baseado em Linux, ele não implementa contas de usuário da mesma maneira que outros sistemas do tipo Unix. No Android, o suporte multi-usuário do kernel Linux é usado para sandbox de aplicativos: com algumas exceções, cada aplicativo é executado como se estivesse sob um usuário Linux separado, efetivamente isolado de outros aplicativos e do resto do sistema operacional.

O arquivo [android_filesystem_config.h](https://android.googlesource.com/platform/system/core/+/master/libcutils/include/private/android_filesystem_config.h) inclui uma lista dos usuários e grupos predefinidos aos quais os processos do sistema são atribuídos. UIDs (userIDs) para outros aplicativos são adicionados conforme estes são instalados.

Por exemplo, o Android 9.0 (API level 28) define os seguintes usuários do sistema:

```c
    #define AID_ROOT             0  /* traditional unix root user */
    #...
    #define AID_SYSTEM        1000  /* system server */
    #...
    #define AID_SHELL         2000  /* adb and debug shell user */
    #...
    #define AID_APP_START          10000  /* first app user */
    ...
```

#### SELinux

Security-Enhanced Linux (SELinux) usa um sistema de Controle de Acesso Mandatório (MAC) para bloquear ainda mais quais processos devem ter acesso a quais recursos. Cada recurso recebe um rótulo na forma de `user:role:type:mls_level` que define quais usuários são capazes de executar quais tipos de ações nele. Por exemplo, um processo pode apenas ser capaz de ler um arquivo, enquanto outro processo pode ser capaz de editar ou excluir o arquivo. Desta forma, trabalhando em um princípio de menor privilégio, processos vulneráveis são mais difíceis de explorar

através de escalação de privilégios ou movimento lateral.

Mais informações estão disponíveis na [documentação do Android](https://source.android.com/security/selinux "Security-Enhanced Linux no Android").

#### Permissões

O Android implementa um sistema extensivo de permissões que é usado como um mecanismo de controle de acesso. Ele garante acesso controlado a dados sensíveis do usuário e recursos do dispositivo. O Android categoriza permissões em diferentes [tipos](https://developer.android.com/guide/topics/permissions/overview#types) oferecendo vários níveis de proteção.

> Antes do Android 6.0 (API level 23), todas as permissões que um aplicativo solicitava eram concedidas na instalação (Permissões de tempo de instalação). A partir do API level 23 em diante, o usuário deve aprovar algumas solicitações de permissões durante o tempo de execução (Permissões de tempo de execução).

Mais informações estão disponíveis na [documentação do Android](https://developer.android.com/guide/topics/permissions/overview) incluindo várias [considerações](https://developer.android.com/training/permissions/evaluating) e [melhores práticas](https://developer.android.com/training/permissions/usage-notes).

Para aprender como testar permissões de aplicativos, consulte a seção [Testando Permissões de Aplicativo](0x05h-Testing-Platform-Interaction.md#app-permissions) no capítulo "APIs da Plataforma Android".

### Segurança de rede

#### TLS por Padrão

Por padrão, desde o Android 9 (API level 28), toda atividade de rede é tratada como sendo executada em um ambiente hostil. Isso significa que o sistema Android permitirá apenas que aplicativos se comuniquem por um canal de rede que é estabelecido usando o protocolo Transport Layer Security (TLS). Este protocolo efetivamente criptografa todo o tráfego de rede e cria um canal seguro para um servidor. Pode ser o caso de você querer usar conexões de tráfego claro por razões de legado. Isso pode ser alcançado adaptando o arquivo `res/xml/network_security_config.xml` no aplicativo.

Mais informações estão disponíveis na [documentação do Android](https://developer.android.com/training/articles/security-config.html).

#### DNS sobre TLS

Suporte system-wide para DNS sobre TLS foi introduzido desde o Android 9 (API level 28). Ele permite que você execute consultas para servidores DNS usando o protocolo TLS. Um canal seguro é estabelecido com o servidor DNS através do qual a consulta DNS é enviada. Isso garante que nenhum dado sensível seja exposto durante uma pesquisa DNS.

Mais informações estão disponíveis no [blog Android Developers](https://android-developers.googleblog.com/2018/04/dns-over-tls-support-in-android-p.html).

### Anti-exploração

#### ASLR, KASLR, PIE e DEP

Address Space Layout Randomization (ASLR), que faz parte do Android desde o Android 4.1 (API level 15), é uma proteção padrão contra ataques de buffer overflow, que garante que tanto o aplicativo quanto o SO sejam carregados para endereços de memória aleatórios, tornando difícil obter o endereço correto para uma região de memória ou biblioteca específica. No Android 8.0 (API level 26), esta proteção também foi implementada para o kernel (KASLR). A proteção ASLR só é possível se o aplicativo pode ser carregado em um lugar aleatório na memória, o que é indicado pelo flag Position Independent Executable (PIE) do aplicativo. Desde o Android 5.0 (API level 21), o suporte para bibliotecas nativas não habilitadas para PIE foi removido. Finalmente, Data Execution Prevention (DEP) previne a execução de código na stack e heap, que também é usado para combater exploits de buffer overflow.

Mais informações estão disponíveis no [blog Android Developers](https://android-developers.googleblog.com/2016/07/protecting-android-with-more-linux.html "Protegendo Android com mais defesas do kernel Linux").

#### Filtro SECCOMP

Aplicativos Android podem conter código nativo escrito em C ou C++. Esses binários compilados podem se comunicar tanto com o Android Runtime através de bindings Java Native Interface (JNI), quanto com o SO através de system calls. Alguns system calls não são implementados, ou não devem ser chamados por aplicativos normais. Como esses system calls se comunicam diretamente com o kernel, eles são um alvo principal para desenvolvedores de exploits. Com o Android 8 (API level 26), o Android introduziu o suporte para filtros Secure Computing (SECCOMP) para todos os processos baseados em Zygote (ou seja, aplicativos de usuário). Esses filtros restringem os syscalls disponíveis àqueles expostos através do bionic.

Mais informações estão disponíveis no [blog Android Developers](https://android-developers.googleblog.com/2017/07/seccomp-filter-in-android-o.html "Filtro Seccomp no Android O").

## Estrutura de Aplicativos Android

### Comunicação com o Sistema Operacional

Aplicativos Android interagem com serviços do sistema através do Android Framework, uma camada de abstração que oferece APIs Java de alto nível. A maioria desses serviços é invocada via chamadas de método Java normais e são traduzidas para chamadas IPC para serviços do sistema que estão sendo executados em segundo plano. Exemplos de serviços do sistema incluem:

- Conectividade (Wi-Fi, Bluetooth, NFC, etc.)
- Arquivos
- Câmeras
- Geolocalização (GPS)
- Microfone

O framework também oferece funções de segurança comuns, como criptografia.

As especificações da API mudam com cada nova versão do Android. Correções críticas de bugs e patches de segurança são geralmente aplicados a versões anteriores também.

[Versões de API](https://developer.android.com/guide/topics/manifest/uses-sdk-element#ApiLevels "O que é nível de API?") notáveis. Veja @MASTG-BEST-0010 para mais informações sobre recursos de segurança e privacidade introduzidos em diferentes versões do Android.

Lançamentos de desenvolvimento do Android seguem uma estrutura única. Eles são organizados em famílias e recebem codinomes alfabéticos inspirados em guloseimas saborosas. Você pode encontrá-los todos [aqui](https://source.android.com/docs/setup/about/build-numbers "Codenames, tags e números de build").

### O Sandbox do Aplicativo

Aplicativos são executados no Android Application Sandbox, que separa os dados do aplicativo e a execução de código de outros aplicativos no dispositivo. Como mencionado antes, esta separação adiciona uma primeira camada de defesa.

A instalação de um novo aplicativo cria um novo diretório nomeado após o pacote do aplicativo, o que resulta no seguinte caminho: `/data/data/[nome-do-pacote]`. Este diretório mantém os dados do aplicativo. Permissões de diretório Linux são definidas de forma que o diretório possa ser lido e escrito apenas com o UID único do aplicativo.

<img src="Images/Chapters/0x05a/Selection_003.png" width="400px" />

Podemos confirmar isso olhando as permissões do sistema de arquivos na pasta `/data/data`. Por exemplo, podemos ver que o Google Chrome e o Calendar são atribuídos um diretório cada e são executados sob diferentes contas de usuário:

```bash
drwx------  4 u0_a97              u0_a97              4096 2017-01-18 14:27 com.android.calendar
drwx------  6 u0_a120             u0_a120             4096 2017-01-19 12:54 com.android.chrome
```

Desenvolvedores que querem que seus aplicativos compartilhem um sandbox comum podem contornar o sandboxing. Quando dois aplicativos são assinados com o mesmo certificado e explicitamente compartilham o mesmo ID de usuário (tendo o _sharedUserId_ em seus arquivos _AndroidManifest.xml_), cada um pode acessar o diretório de dados do outro. Veja o seguinte exemplo para alcançar isso no aplicativo NFC:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.android.nfc"
  android:sharedUserId="android.uid.nfc">
```

#### Gerenciamento de Usuários Linux

O Android aproveita o gerenciamento de usuários Linux para isolar aplicativos. Esta abordagem é diferente do uso de gerenciamento de usuários em ambientes Linux tradicionais, onde múltiplos aplicativos são frequentemente executados pelo mesmo usuário. O Android cria um UID único para cada aplicativo Android e executa o aplicativo em um processo separado. Consequentemente, cada aplicativo pode acessar apenas seus próprios recursos. Esta proteção é aplicada pelo kernel Linux.

Geralmente, aplicativos recebem UIDs na faixa de 10000 e 99999. Aplicativos Android recebem um nome de usuário baseado em seu UID. Por exemplo, o aplicativo com UID 10188 recebe o nome de usuário `u0_a188`. Se as permissões que um aplicativo solicitou são concedidas, o ID do grupo correspondente é adicionado ao processo do aplicativo. Por exemplo, o ID do usuário do aplicativo abaixo é 10188. Ele pertence ao ID do grupo 3003 (inet). Esse grupo está relacionado à permissão android.permission.INTERNET. A saída do comando `id` é mostrada abaixo.

```bash
$ id
uid=10188(u0_a188) gid=10188(u0_a188) groups=10188(u0_a188),3003(inet),
9997(everybody),50188(all_a188) context=u:r:untrusted_app:s0:c512,c768
```

A relação entre IDs de grupo e permissões é definida no seguinte arquivo:

[platform.xml](https://android.googlesource.com/platform/frameworks/base/+/master/data/etc/platform.xml)

```xml
<permission name="android.permission.INTERNET" >
    <group gid="inet" />
</permission>

<permission name="android.permission.READ_LOGS" >
    <group gid="log" />
</permission>

<permission name="android.permission.WRITE_MEDIA_STORAGE" >
    <group gid="media_rw" />
    <group gid="sdcard_rw" />
</permission>
```

#### Zygote

O processo `Zygote` inicia durante a [inicialização do Android](https://github.com/dogriffiths/HeadFirstAndroid/wiki/How-Android-Apps-are-Built-and-Run "Como Aplicativos Android são Construídos e Executados"). Zygote é um serviço do sistema para lançar aplicativos. O processo Zygote é um processo "base" que contém todas as bibliotecas principais que o aplicativo precisa. Ao ser iniciado, o Zygote abre o socket `/dev/socket/zygote` e escuta por conexões de clientes locais. Quando recebe uma conexão, ele faz fork de um novo processo, que então carrega e executa o código específico do aplicativo.

#### Ciclo de Vida do Aplicativo

No Android, o tempo de vida de um processo de aplicativo é controlado pelo sistema operacional. Um novo processo Linux é criado quando um componente do aplicativo é iniciado e o mesmo aplicativo ainda não tem nenhum outro componente em execução. O Android pode matar este processo quando o último não for mais necessário ou quando a recuperação de memória for necessária para executar aplicativos mais importantes. A decisão de matar um processo está primariamente relacionada ao estado da interação do usuário com o processo. Em geral, processos podem estar em um de quatro estados.

- Um processo em primeiro plano (por exemplo, uma atividade em execução no topo da tela ou um BroadcastReceiver em execução)
- Um processo visível é um processo do qual o usuário está ciente, então matá-lo teria um impacto negativo perceptível na experiência do usuário. Um exemplo é executar uma atividade que é visível para o usuário na tela, mas não em primeiro plano.

- Um processo de serviço é um processo que hospeda um serviço que foi iniciado com o método `startService`. Embora esses processos não sejam diretamente visíveis para o usuário, eles são geralmente coisas com as quais o usuário se importa (como upload ou download de dados de rede em segundo plano), então o sistema sempre manterá tais processos em execução, a menos que haja memória insuficiente para reter todos os processos em primeiro plano e visíveis.
- Um processo em cache é um processo que não é atualmente necessário, então o sistema é livre para matá-lo quando a memória for necessária.
Aplicativos devem implementar métodos de callback que reagem a uma série de eventos; por exemplo, o handler `onCreate` é chamado quando o processo do aplicativo é primeiro criado. Outros métodos de callback incluem `onLowMemory`, `onTrimMemory` e `onConfigurationChanged`.

### Pacotes de Aplicativos

Aplicativos Android podem ser enviados em duas formas: o arquivo Android Package Kit (APK) ou um [Android App Bundle](https://developer.android.com/guide/app-bundle "Android App Bundle") (.aab). Android App Bundles fornecem todos os recursos necessários para um aplicativo, mas adiam a geração do APK e sua assinatura para o Google Play. App Bundles são binários assinados que contêm o código do aplicativo em vários módulos. O módulo base contém o núcleo do aplicativo. O módulo base pode ser estendido com vários módulos que contêm novos enriquecimentos/funcionalidades para o aplicativo, conforme explicado mais detalhadamente na [documentação do desenvolvedor para app bundle](https://developer.android.com/guide/app-bundle "Documentação sobre App Bundle").
Se você tem um Android App Bundle, pode melhor usar a ferramenta de linha de comando [bundletool](https://developer.android.com/studio/command-line/bundletool "bundletool") do Google para construir APKs não assinados a fim de usar as ferramentas existentes no APK. Você pode criar um APK a partir de um arquivo AAB executando o seguinte comando:

```bash
bundletool build-apks --bundle=/MyApp/my_app.aab --output=/MyApp/my_app.apks
```

Se você quiser criar APKs assinados prontos para implantação em um dispositivo de teste, use:

```bash
$ bundletool build-apks --bundle=/MyApp/my_app.aab --output=/MyApp/my_app.apks
--ks=/MyApp/keystore.jks
--ks-pass=file:/MyApp/keystore.pwd
--ks-key-alias=MyKeyAlias
--key-pass=file:/MyApp/key.pwd
```

Recomendamos que você teste tanto o APK com quanto sem os módulos adicionais, para que fique claro se os módulos adicionais introduzem e/ou corrigem problemas de segurança para o módulo base.

### Android Manifest

Cada aplicativo Android contém um arquivo `AndroidManifest.xml` na raiz do APK, armazenado em formato binário XML. Este arquivo define a estrutura do aplicativo e as propriedades-chave usadas pelo sistema operacional Android durante a instalação e tempo de execução.

Elementos relevantes para segurança incluem:

- **Permissões:** Declara permissões necessárias usando `<uses-permission>` como acesso à internet, câmera, armazenamento, localização ou contatos. Estes definem os limites de acesso do aplicativo e devem seguir o princípio do menor privilégio. Permissões personalizadas podem ser definidas usando `<permission>` e devem incluir um `protectionLevel` adequado, como `signature` ou `dangerous` para evitar serem mal utilizadas por outros aplicativos.
- **Componentes:** O manifesto lista todos os [componentes do aplicativo](#componentes-do-aplicativo) declarados no aplicativo servindo como pontos de entrada. Eles podem ser expostos a outros aplicativos (via filtros de intent ou o atributo `exported`) então são críticos para determinar como um invasor pode interagir com o aplicativo. Os principais tipos de componentes são:
    - **Activities:** definem telas de interface do usuário.
    - **Services:** executam tarefas em segundo plano.
    - **Broadcast Receivers:** lidam com mensagens externas.
    - **Content Providers:** expõem dados estruturados.
- **Deep Links:** [Deep links](0x05h-Testing-Platform-Interaction.md#deep-links) são configurados via filtros de intent com a ação `VIEW`, categoria `BROWSABLE` e um elemento `data` especificando um padrão URI. Estes podem expor atividades para links web ou de aplicativos e devem ser verificados cuidadosamente para evitar riscos de injeção ou spoofing. Adicionar `android:autoVerify="true"` habilita App Links, que restringe o tratamento de links verificados ao aplicativo declarado, reduzindo o risco de sequestro de link.
- **Usa Tráfego em Texto Claro:** O atributo `android:usesCleartextTraffic` controla se o aplicativo permite tráfego HTTP não criptografado. A partir do Android 9 (API 28) em diante, o tráfego em texto claro

é desabilitado por padrão, a menos que explicitamente permitido. Este atributo também pode ser substituído pelo `networkSecurityConfig`.
- **Configuração de Segurança de Rede:** Um arquivo XML opcional definido via `android:networkSecurityConfig`, disponível desde o Android 7.0 (API level 24), que fornece controle granular sobre o [comportamento de segurança de rede](0x05g-Testing-Network-Communication.md#android-network-security-configuration). Ele permite especificar autoridades certificadoras confiáveis, requisitos TLS por domínio e exceções de tráfego em texto claro, substituindo configurações globais definidas em `android:usesCleartextTraffic`.
- **Comportamento de Backup:** O atributo `android:allowBackup` permite ou impede que dados do aplicativo sejam [backupados](0x05d-Testing-Data-Storage.md#backups).
- **Afinidades de Tarefa e Modos de Inicialização:** Essas configurações influenciam como as atividades são agrupadas e iniciadas. Configurações incorretas podem permitir sequestro de tarefa ou ataques de estilo phishing se o aplicativo de um invasor imitar componentes legítimos.

A lista completa de opções de manifesto disponíveis pode ser encontrada na [documentação oficial do arquivo Android Manifest](https://developer.android.com/guide/topics/manifest/manifest-intro.html "Guia do Desenvolvedor Android para Manifest").

No tempo de build, o manifesto é mesclado com aqueles de todas as bibliotecas e dependências incluídas. O manifesto mesclado final pode incluir permissões, componentes ou configurações adicionais não declaradas explicitamente pelo desenvolvedor. Revisões de segurança devem analisar a saída mesclada para entender a exposição real do aplicativo.

Aqui está um exemplo de um arquivo de manifesto conforme definido por um desenvolvedor. Ele declara várias permissões, permite backup e define a atividade principal do aplicativo:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_CONTACTS" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.MASTestApp"
        tools:targetApi="31">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:theme="@style/Theme.MASTestApp">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

Se você obtivesse o arquivo AndroidManifest.xml de um APK (@MASTG-TECH-0117), você veria que ele inclui elementos adicionais, como o atributo `package`, que define o identificador único do aplicativo, o elemento `<uses-sdk>` que especifica o `android:minSdkVersion` e `android:targetSdkVersion`, novas atividades, providers e receivers e outros atributos como `android:debuggable="true"` que indica que o aplicativo está em modo de depuração.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" android:versionCode="1" android:versionName="1.0"
    android:compileSdkVersion="35"
    android:compileSdkVersionCodename="15"
    package="org.owasp.mastestapp"
    platformBuildVersionCode="35"
    platformBuildVersionName="15">
    <uses-sdk
        android:minSdkVersion="29"
        android:targetSdkVersion="35"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_CONTACTS"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <permission
        android:name="org.owasp.mastestapp.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION"
        android:protectionLevel="signature"/>
    <uses-permission android:name="org.owasp.mastestapp.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION"/>
    <application
        android:theme="@style/Theme.MASTestApp"
        android:label="@string/app_name"
        android:icon="@mipmap/ic_launcher"
        android:debuggable="true"
        android:testOnly="true"
        android:allowBackup="true"
        android:supportsRtl="true"
        android:extractNativeLibs="false"
        android:fullBackupContent="@xml/backup_rules"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:appComponentFactory="androidx.core.app.CoreComponentFactory"
        android:dataExtractionRules="@xml/data_extraction_rules">
        <activity
            android:theme="@style/Theme.MASTestApp"
            android:name="org.owasp.mastestapp.MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
        <activity
            android:name="androidx.compose.ui.tooling.PreviewActivity"
            android:exported="true"/>
        <activity
            android:name="androidx.activity.ComponentActivity"
            android:exported="true"/>
        <provider
            android:name="androidx.startup.InitializationProvider"
            android:exported="false"
            android:authorities="org.owasp.mastestapp.androidx-startup">
            <meta-data
                android:name="androidx.emoji2.text.EmojiCompatInitializer"
                android:value="androidx.startup"/>
            ...
        </provider>
        <receiver
            android:name="androidx.profileinstaller.ProfileInstallReceiver"
            android:permission="android.permission.DUMP"
            android:enabled="true"
            android:exported="true"
            android:directBootAware="false">
            <intent-filter>
                <action android:name="androidx.profileinstaller.action.INSTALL_PROFILE"/>
            </intent-filter>
            ...
        </receiver>
    </application>
</manifest>
```

### Componentes do Aplicativo

Aplicativos Android são feitos de vários componentes de alto nível. Os principais componentes são:

- Activities
- Fragments
- Intents
- Broadcast receivers
- Content providers e services

Todos esses elementos são fornecidos pelo sistema operacional Android, na forma de classes predefinidas disponíveis através de APIs.

#### Activities

Activities compõem a parte visível de qualquer aplicativo. Há uma atividade por tela, então um aplicativo com três telas diferentes implementa três atividades diferentes. Activities são declaradas estendendo a classe Activity. Elas contêm todos os elementos de interface do usuário: fragments, views e layouts.

Cada atividade precisa ser declarada no Android Manifest com a seguinte sintaxe:

```xml
<activity android:name="ActivityName">
</activity>
```

Activities não declaradas no manifesto não podem ser exibidas, e tentar lançá-las levantará uma exceção.

Como aplicativos, atividades têm seu próprio ciclo de vida e precisam monitorar mudanças do sistema para lidar com elas. Activities podem estar nos seguintes estados: ativa, pausada, parada e inativa. Esses estados são gerenciados pelo sistema operacional Android. Consequentemente, atividades podem implementar os seguintes gerenciadores de eventos:

- onCreate
- onSaveInstanceState
- onStart
- onResume
- onRestoreInstanceState
- onPause
- onStop
- onRestart
- onDestroy

Um aplicativo pode não implementar explicitamente todos os gerenciadores de eventos, caso em que ações padrão são tomadas. Tipicamente, pelo menos o gerenciador `onCreate` é sobrescrito pelos desenvolvedores do aplicativo. É assim que a maioria dos componentes de interface do usuário são declarados e inicializados. `onDestroy` pode ser sobrescrito quando recursos (como conexões de rede ou conexões com bancos de dados) devem ser explicitamente liberados ou ações específicas devem ocorrer quando o aplicativo é encerrado.

#### Fragments

Um fragment representa um comportamento ou uma porção da interface do usuário dentro da atividade. Fragments foram introduzidos no Android com a versão Honeycomb 3.0 (API level 11).

Fragments são destinados a encapsular partes da interface para facilitar a reutilização e adaptação a diferentes tamanhos de tela. Fragments são entidades autônomas no sentido de que incluem todos os seus componentes necessários (eles têm seu próprio layout, botões, etc.). No entanto, eles devem ser integrados com atividades para serem úteis: fragments não podem existir por conta própria. Eles têm seu próprio ciclo de vida, que está vinculado ao ciclo de vida das Activities que os implementam.

Como fragments têm seu próprio ciclo de vida, a classe Fragment contém gerenciadores de eventos que podem ser redefinidos e estendidos. Esses gerenciadores de eventos incluem onAttach, onCreate, onStart, onDestroy e onDetach. Vários outros existem; o leitor deve se referir à [especificação Android Fragment](https://developer.android.com/guide/components/fragments "Classe Fragment") para mais detalhes.

Fragments podem ser facilmente implementados estendendo a classe Fragment fornecida pelo Android:

Exemplo em Java:

```java
public class MyFragment extends Fragment {
    ...
}
```

Exemplo em Kotlin:

```kotlin
class MyFragment : Fragment() {
    ...
}
```

Fragments não precisam ser declarados em arquivos de manifesto porque dependem de atividades.

Para gerenciar seus fragments, uma atividade pode usar um Gerenciador de Fragment (classe FragmentManager). Esta classe facilita encontrar, adicionar, remover e substituir fragments associados.

Gerenciadores de Fragment podem ser criados via o seguinte:

Exemplo em Java:

```java
FragmentManager fm = getFragmentManager();
```

Exemplo em Kotlin:

```kotlin
var fm = fragmentManager
```

Fragments não necessariamente têm uma interface de usuário; eles podem ser uma maneira conveniente e eficiente de gerenciar operações em segundo plano pertinentes à interface do usuário do aplicativo. Um fragment pode ser declarado persistente para que o sistema preserve seu estado mesmo se sua Activity for destruída.

#### Content Providers

O Android usa SQLite para armazenar dados permanentemente: como no Linux, os dados são armazenados em arquivos. SQLite é uma tecnologia de armazenamento de dados relacional leve, eficiente e de código aberto que não requer muito poder de processamento, o que a torna ideal para uso móvel. Uma API inteira com classes específicas (Cursor, ContentValues, SQLiteOpenHelper, ContentProvider, ContentResolver, etc.) está disponível.
SQLite não é executado como um processo separado; é parte do aplicativo.
Por padrão, um banco de dados pertencente a um determinado aplicativo é acessível apenas a este aplicativo. No entanto, content providers oferecem um grande mecanismo para abstrair fontes de dados (incluindo bancos de dados e arquivos planos); eles também fornecem um mecanismo padrão e eficiente para compartilhar dados entre aplicativos, incluindo aplicativos nativos. Para ser acessível a outros aplicativos, um content provider precisa ser explicitamente declarado no arquivo de manifesto do aplicativo que o compartilhará. Desde que content providers não sejam declarados, eles não serão exportados e só poderão ser chamados pelo aplicativo que os cria.

Content providers são implementados através de um esquema de endereçamento URI: todos usam o modelo content://. Independentemente do tipo de fontes (banco de dados SQLite, arquivo plano, etc.), o esquema de endereçamento é sempre o mesmo, abstraindo assim as fontes e oferecendo ao desenvolvedor um esquema único. Content providers oferecem todas as operações regulares de banco de dados: criar, ler, atualizar, excluir. Isso significa que qualquer aplicativo com direitos adequados em seu arquivo de manifesto pode manipular os dados de outros aplicativos.

#### Services

Services são componentes do SO Android (baseados na classe Service) que executam tarefas em segundo plano (processamento de dados, iniciando intents e notificações, etc.) sem apresentar uma interface de usuário. Services são destinados a executar processos de longo prazo. Suas prioridades do sistema são menores que

aquelas de aplicativos ativos e maiores que aquelas de aplicativos inativos. Portanto, é menos provável que sejam mortos quando o sistema precisa de recursos, e podem ser configurados para reiniciar automaticamente quando recursos suficientes se tornarem disponíveis. Isso torna os services um ótimo candidato para executar tarefas em segundo plano. Observe que Services, como Activities, são executados na thread principal do aplicativo. Um service não cria sua própria thread e não é executado em um processo separado, a menos que você especifique o contrário.

### Comunicação Inter-Processo

Como já aprendemos, todo processo Android tem seu próprio espaço de endereço sandboxado. Instalações de comunicação inter-processo permitem que aplicativos troquem sinais e dados com segurança. Em vez de depender das instalações IPC padrão do Linux, o IPC do Android é baseado no Binder, uma implementação personalizada do OpenBinder. A maioria dos serviços do sistema Android e todos os serviços IPC de alto nível dependem do Binder.

O termo _Binder_ significa muitas coisas diferentes, incluindo:

- Binder Driver: o driver em nível de kernel
- Binder Protocol: protocolo baseado em ioctl de baixo nível usado para se comunicar com o driver binder
- Interface IBinder: um comportamento bem definido que objetos Binder implementam
- Objeto Binder: implementação genérica da interface IBinder
- Serviço Binder: implementação do objeto Binder; por exemplo, serviço de localização e serviço de sensor
- Cliente Binder: um objeto usando o serviço Binder

O framework Binder inclui um modelo de comunicação cliente-servidor. Para usar IPC, aplicativos chamam métodos IPC em objetos proxy. Os objetos proxy transparentemente _empacotam_ os parâmetros de chamada em um _parcel_ e enviam uma transação para o servidor Binder, que é implementado como um driver de caractere (/dev/binder). O servidor mantém um pool de threads para lidar com solicitações de entrada e entrega mensagens para o objeto de destino. Da perspectiva do aplicativo cliente, tudo isso parece uma chamada de método regular, todo o trabalho pesado é feito pelo framework Binder.

<img src="Images/Chapters/0x05a/binder.jpg" width="400px" />

- _Visão Geral do Binder - Fonte da imagem: [Android Binder por Thorsten Schreiber](https://1library.net/document/z33dd47z-android-android-interprocess-communication-thorsten-schreiber-somorovsky-bussmeyer.html "Android Binder")_

Serviços que permitem que outros aplicativos se liguem a eles são chamados _serviços vinculados_. Esses serviços devem fornecer uma interface IBinder para clientes. Desenvolvedores usam a Android Interface Descriptor Language (AIDL) para escrever interfaces para serviços remotos.

ServiceManager é um daemon do sistema que gerencia o registro e a pesquisa de serviços do sistema. Ele mantém uma lista de pares nome/Binder para todos os serviços registrados. Serviços são adicionados com `addService` e recuperados por nome com o método estático `getService` em `android.os.ServiceManager`:

Exemplo em Java:

```java
public static IBinder getService(String name) {
        try {
            IBinder service = sCache.get(name);
            if (service != null) {
                return service;
            } else {
                return getIServiceManager().getService(name);
            }
        } catch (RemoteException e) {
            Log.e(TAG, "error in getService", e);
        }
        return null;
    }
```

Exemplo em Kotlin:

```kotlin
companion object {
        private val sCache: Map<String, IBinder> = ArrayMap()
        fun getService(name: String): IBinder? {
            try {
                val service = sCache[name]
                return service ?: getIServiceManager().getService(name)
            } catch (e: RemoteException) {
                Log.e(FragmentActivity.TAG, "error in getService", e)
            }
            return null
        }
    }
```

Você pode consultar a lista de serviços do sistema com o comando `service list`.

```bash
$ adb shell service list
Found 99 services:
0 carrier_config: [com.android.internal.telephony.ICarrierConfigLoader]
1 phone: [com.android.internal.telephony.ITelephony]
2 isms: [com.android.internal.telephony.ISms]
3 iphonesubinfo: [com.android.internal.telephony.IPhoneSubInfo]
```

#### Intents

_Mensagens de intent_ são um framework de comunicação assíncrona construído sobre o Binder. Este framework permite tanto mensagens ponto-a-ponto quanto publicação-assinatura. Um _Intent_ é um objeto de mensagem que pode ser usado para solicitar uma ação de outro componente do aplicativo. Embora intents facilitem a comunicação inter-componente de várias maneiras, há três casos de uso fundamentais:

- Iniciando uma atividade
    - Uma atividade representa uma única tela em um aplicativo. Você pode iniciar uma nova instância de uma atividade passando um intent para `startActivity`. O intent descreve a atividade e carrega dados necessários.
- Iniciando um serviço
    - Um Service é um componente que executa operações em segundo plano, sem uma interface de usuário. Com Android 5.0 (API level 21) e posterior, você pode iniciar um serviço com JobScheduler.
- Entregando um broadcast
    - Um broadcast é uma mensagem que qualquer aplicativo pode receber. O sistema entrega broadcasts para eventos do sistema, incluindo inicialização do sistema e inicialização de carregamento. Você pode entregar um broadcast para outros aplicativos passando um intent para `sendBroadcast` ou `sendOrderedBroadcast`.

Há dois tipos de intents. Intents explícitos nomeiam o componente que será iniciado (o nome de classe totalmente qualificado). Por exemplo:

Exemplo em Java:

```java
Intent intent = new Intent(this, myActivity.myClass);
```

Exemplo em Kotlin:

```kotlin
var intent = Intent(this, myActivity.myClass)
```

Intents implícitos são enviados para o SO para executar uma determinada ação em um determinado conjunto de dados (A URL do site OWASP em nosso exemplo abaixo). Cabe ao sistema decidir qual aplicativo ou classe executará o serviço correspondente. Por exemplo:

Exemplo em Java:

```java
Intent intent = new Intent(Intent.MY_ACTION, Uri.parse("https://www.owasp.org"));
```

Exemplo em Kotlin:

```kotlin
var intent = Intent(Intent.MY_ACTION, Uri.parse("https://www.owasp.org"))
```

Um _filtro de intent_ é uma expressão em arquivos Android Manifest que especifica o tipo de intents que o componente gostaria de receber. Por exemplo, declarando um filtro de intent para uma atividade, você torna possível que outros aplicativos iniciem diretamente sua atividade com um certo tipo de intent. Da mesma forma, sua atividade só pode ser iniciada com um intent explícito se você não declarar nenhum filtro de intent para ela.

O Android usa intents para transmitir mensagens para aplicativos (como uma chamada recebida ou SMS), informações importantes de alimentação (bateria fraca, por exemplo) e mudanças de rede (perda de conexão, por exemplo). Dados extras podem ser adicionados a intents (através de `putExtra`/`getExtras`).

Aqui está uma pequena lista de intents enviados pelo sistema operacional. Todas as constantes são definidas na classe Intent, e a lista completa está na documentação oficial do Android:

- ACTION_CAMERA_BUTTON
- ACTION_MEDIA_EJECT
- ACTION_NEW_OUTGOING_CALL
- ACTION_TIMEZONE_CHANGED

Para melhorar a segurança e privacidade, um Local Broadcast Manager é usado para enviar e receber intents dentro de um aplicativo sem tê-los enviados para o resto do sistema operacional. Isso é muito útil para garantir que dados sensíveis e privados não saiam do perímetro do aplicativo (dados de geolocalização, por exemplo).

#### Broadcast Receivers

Broadcast Receivers são componentes que permitem que aplicativos recebam notificações de outros aplicativos e do próprio sistema. Com eles, aplicativos podem reagir a eventos (internos, iniciados por outros aplicativos ou iniciados pelo sistema operacional). Eles são geralmente usados para atualizar interfaces do usuário, iniciar serviços, atualizar conteúdo e criar notificações de usuário.

Há duas maneiras de tornar um Broadcast Receiver conhecido para o sistema. Uma maneira é declará-lo no arquivo Android Manifest. O manifesto deve especificar uma associação entre o Broadcast Receiver e um filtro de intent para indicar as ações que o receiver deve escutar.

Uma declaração de exemplo de Broadcast Receiver com um filtro de intent em um manifesto:

```xml
<receiver android:name=".MyReceiver" >
    <intent-filter>
        <action android:name="com.owasp.myapplication.MY_ACTION" />
    </intent-filter>
</receiver>
```

Observe que neste exemplo, o Broadcast Receiver não inclui o atributo [`android:exported`](https://developer.android.com/guide/topics/manifest/receiver-element "elemento receiver"). Como pelo menos um filtro foi definido, o valor padrão será definido como "true". Na ausência de quaisquer filtros, será definido como "false".

A outra maneira é criar o receiver dinamicamente em código. O receiver pode então se registrar com o método [`Context.registerReceiver`](https://developer.android.com/reference/android/content/Context.html#registerReceiver%28android.content.BroadcastReceiver,%2520android.content.IntentFilter%29 "Context.registerReceiver").

Um exemplo de registro de um Broadcast Receiver dinamicamente:

Exemplo em Java:

```java
// Define a broadcast receiver
BroadcastReceiver myReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.d(TAG, "Intent received by myReceiver");
    }
};
// Define an intent filter with actions that the broadcast receiver listens for
IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction("com.owasp.myapplication.MY_ACTION");
// To register the broadcast receiver
registerReceiver(myReceiver, intentFilter);
// To un-register the broadcast receiver
unregisterReceiver(myReceiver);
```

Exemplo em Kotlin:

```kotlin
// Define a broadcast receiver
val myReceiver: BroadcastReceiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        Log.d(FragmentActivity.TAG, "Intent received by myReceiver")
    }
}
// Define an intent filter with actions that the broadcast receiver listens for
val intentFilter = IntentFilter()
intentFilter.addAction("com.owasp.myapplication.MY_ACTION")
// To register the broadcast receiver
registerReceiver(myReceiver, intentFilter)
// To un-register the broadcast receiver
unregisterReceiver(myReceiver)
```

Note que o sistema inicia um aplicativo com o receiver registrado automaticamente quando um intent relevante é levantado.

De acordo com [Visão Geral de Broadcasts](https://developer.android.com/guide/components/broadcasts "Visão Geral de Broadcasts"), um broadcast é considerado "implícito" se não t

como alvo um aplicativo especificamente. Depois de receber um broadcast implícito, o Android listará todos os aplicativos que registraram uma determinada ação em seus filtros. Se mais de um aplicativo tiver registrado para a mesma ação, o Android solicitará que o usuário selecione na lista de aplicativos disponíveis.

Uma característica interessante dos Broadcast Receivers é que eles podem ser priorizados; desta forma, um intent será entregue a todos os receivers autorizados de acordo com sua prioridade. Uma prioridade pode ser atribuída a um filtro de intent no manifesto através do atributo `android:priority`, bem como programaticamente via o método [`IntentFilter.setPriority`](https://developer.android.com/reference/android/content/IntentFilter#setPriority%28int%29 "IntentFilter.setPriority"). No entanto, observe que receivers com a mesma prioridade serão [executados em uma ordem arbitrária](https://developer.android.com/guide/components/broadcasts.html#sending-broadcasts "Enviando Broadcasts").

Se seu aplicativo não deve enviar broadcasts entre aplicativos, use um Local Broadcast Manager ([`LocalBroadcastManager`](https://developer.android.com/reference/androidx/localbroadcastmanager/content/LocalBroadcastManager.html "LocalBroadcastManager")). Eles podem ser usados para garantir que intents sejam recebidos apenas do aplicativo interno, e qualquer intent de qualquer outro aplicativo será descartado. Isso é muito útil para melhorar a segurança e a eficiência do aplicativo, pois nenhuma comunicação interprocesso está envolvida. No entanto, observe que a classe `LocalBroadcastManager` está [obsoleta](https://developer.android.com/reference/androidx/localbroadcastmanager/content/LocalBroadcastManager.html "LocalBroadcastManager") e o Google recomenda usar alternativas como [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData.html "LiveData").

Para mais considerações de segurança sobre Broadcast Receiver, consulte [Considerações de Segurança e Melhores Práticas](https://developer.android.com/guide/components/broadcasts.html#security-and-best-practices "Considerações de Segurança e Melhores Práticas").

#### Limitação de Broadcast Receiver Implícito

De acordo com [Otimizações em Segundo Plano](https://developer.android.com/topic/performance/background-optimization "Otimizações em Segundo Plano"), aplicativos direcionando Android 7.0 (API level 24) ou superior não recebem mais broadcast `CONNECTIVITY_ACTION`, a menos que registrem seus Broadcast Receivers com `Context.registerReceiver()`. O sistema também não envia broadcasts `ACTION_NEW_PICTURE` e `ACTION_NEW_VIDEO`.

De acordo com [Limites de Execução em Segundo Plano](https://developer.android.com/about/versions/oreo/background.html#broadcasts "Limites de Execução em Segundo Plano"), aplicativos que direcionam Android 8.0 (API level 26) ou superior não podem mais registrar Broadcast Receivers para broadcasts implícitos em seu manifesto, exceto para aqueles listados em [Exceções de Broadcast Implícito](https://developer.android.com/guide/components/broadcast-exceptions "Exceções de Broadcast Implícito"). Os Broadcast Receivers criados em tempo de execução chamando `Context.registerReceiver` não são afetados por esta limitação.

De acordo com [Mudanças em Broadcasts do Sistema](https://developer.android.com/guide/components/broadcasts#changes-system-broadcasts "Mudanças em Broadcasts do Sistema"), começando com Android 9 (API level 28), o broadcast `NETWORK_STATE_CHANGED_ACTION` não recebe informações sobre a localização do usuário ou dados pessoalmente identificáveis.

## Publicação de Aplicativos Android

Uma vez que um aplicativo foi desenvolvido com sucesso, o próximo passo é publicá-lo e compartilhá-lo com outros. No entanto, aplicativos não podem simplesmente ser adicionados a uma loja e compartilhados, eles devem primeiro ser assinados. A assinatura criptográfica serve como uma marca verificável colocada pelo desenvolvedor do aplicativo. Ela identifica o autor do aplicativo e garante que o aplicativo não foi modificado desde sua distribuição inicial.

### Processo de Assinatura

Durante o desenvolvimento, aplicativos são assinados com um certificado gerado automaticamente. Este certificado é inerentemente inseguro e é apenas para depuração. A maioria das lojas não aceita este tipo de certificado para publicação; portanto, um certificado com recursos mais seguros deve ser criado.
Quando um aplicativo é instalado no dispositivo Android, o Gerenciador de Pacotes garante que ele foi assinado com o certificado incluído no APK correspondente. Se a chave pública do certificado corresponder à chave usada para assinar qualquer outro APK no dispositivo, o novo APK pode compartilhar um UID com o APK pré-existente. Isso facilita interações entre aplicativos de um único fornecedor. Alternativamente, especificar permissões de segurança para o nível de proteção Signature é possível; isso restringirá o acesso a aplicativos que foram assinados com a mesma chave.

### Esquemas de Assinatura APK

O Android suporta múltiplos esquemas de assinatura de aplicativos:

- **Abaixo do Android 7.0 (API level 24)**: aplicativos podem usar apenas o esquema de assinatura JAR (v1) que não protege todas as partes do APK. Este esquema é considerado inseguro.
- **Android 7.0 (API level 24) e acima**: aplicativos podem usar o **esquema de assinatura v2**, que assina o APK como um todo, fornecendo proteção mais forte comparada ao método de assinatura v1 (JAR) mais antigo.
- **Android 9 (API level 28) e acima**: É recomendado usar tanto o **esquema de assinatura v2 quanto v3**. O esquema v3 suporta **rotação de chaves**, permitindo que desenvolvedores substituam chaves no evento de um comprometimento sem invalidar assinaturas antigas.
- **Android 11 (API level 30) e acima**: aplicativos podem opcionalmente incluir o **esquema de assinatura v4** para permitir atualizações incrementais mais rápidas.

Para compatibilidade com versões anteriores, um APK pode ser assinado com múltiplos esquemas de assinatura para fazer o aplicativo executar em versões SDK mais novas e mais antigas. Por exemplo, [plataformas mais antigas ignoram assinaturas v2 e verificam apenas assinaturas v1](https://source.android.com/security/apksigning/).

#### Assinatura JAR (Esquema v1)

A versão original de assinatura de aplicativo implementa o APK assinado como um JAR assinado padrão, que deve conter todas as entradas em `META-INF/MANIFEST.MF`. Todos os arquivos devem ser assinados com um certificado comum. Este esquema não protege algumas partes do APK, como metadados ZIP. A desvantagem deste esquema é que o verificador APK precisa processar estruturas de dados não confiáveis antes de aplicar a assinatura, e o verificador descarta as estruturas de dados que não cobrem. Além disso, o verificador APK deve descomprimir todos os arquivos comprimidos, o que leva tempo e memória consideráveis.

Este esquema de assinatura é considerado inseguro, ele é afetado, por exemplo, pela **vulnerabilidade Janus (CVE-2017-13156)**, que pode permitir que atores maliciosos modifiquem arquivos APK sem invalidar a assinatura v1. Como tal, **v1 nunca deve ser confiado para dispositivos executando Android 7.0 e acima**.

#### Esquema de Assinatura APK (Esquema v2)

Com o esquema de assinatura APK, o APK completo é hasheado e assinado, e um Bloco de Assinatura APK é criado e inserido no APK. Durante a validação, o esquema v2 verifica as assinaturas de todo o arquivo APK. Esta forma de verificação APK é mais rápida e oferece proteção mais abrangente contra modificação. Você pode ver o [processo de verificação de assinatura APK para o Esquema v2](https://source.android.com/security/apksigning/v2#verification "Processo de verificação de assinatura APK") abaixo.

<img src="Images/Chapters/0x05a/apk-validation-process.png" width="400px" />

#### Esquema de Assinatura APK (Esquema v3)

O formato de Bloco de Assinatura APK v3 é o mesmo do v2. V3 adiciona informações sobre as versões SDK suportadas e uma estrutura de prova de rotação ao bloco de assinatura APK. No Android 9 (API level 28) e superior, APKs podem ser verificados de acordo com o Esquema de Assinatura APK v3, v2 ou v1. Plataformas mais antigas ignoram assinaturas v3 e tentam verificar a assinatura v2 e depois v1.

O atributo de prova de rotação nos dados assinados do bloco de assinatura consiste em uma lista vinculada individualmente, com cada nó contendo um certificado de assinatura usado para assinar versões anteriores do aplicativo. Para fazer a compatibilidade com versões anteriores funcionar, os certificados de assinatura antigos assinam o novo conjunto de certificados, fornecendo assim a cada nova chave evidência de que deve ser tão confiável quanto a(s) chave(s) mais antiga(s).
Não é mais possível assinar APKs independentemente, porque a estrutura de prova de rotação deve ter os certificados de assinatura antigos assinando o novo conjunto de certificados, em vez de assiná-los um por um. Você pode ver o [processo de verificação do esquema de assinatura APK v3](https://source.android.com/security/apksigning/v3 "Processo de verificação do esquema de assinatura APK v3") abaixo.

<img src="Images/Chapters/0x05a/apk-validation-process-v3-scheme.png" width="400px" />

#### Esquema de Assinatura APK (Esquema v4)

O Esquema de Assinatura APK v4 foi introduzido junto com o Android 11 (API level 30) e requer que todos os dispositivos lançados com Android 11 e superior tenham [fs-verity](https://www.kernel.org/doc/html/latest/filesystems/fsverity.html) habilitado por padrão. fs-verity é um recurso do kernel Linux que é usado principalmente para autenticação de arquivo (detecção de modificações maliciosas) devido ao seu cálculo de hash de arquivo extremamente eficiente. Solicitações de leitura só terão sucesso se o conteúdo verificar contra certificados digitais confiáveis que foram carregados para o keyring do kernel durante o tempo de inicialização.

A assinatura v4 requer uma assinatura complementar v2 ou v3 e, em contraste com esquemas de assinatura anteriores, a assinatura v4 é armazenada em um arquivo separado `<apk name>.apk.idsig`. Lembre-se de especificá-lo usando o flag `--v4-signature-file` ao verificar um APK assinado v4 com `apksigner verify`.

Você pode encontrar informações mais detalhadas na [documentação do desenvolvedor Android](https://source.android.com/security/apksigning/v4).

#### Criando Seu Certificado

O Android usa certificados públicos/privados para assinar aplicativos Android (arquivos .apk). Certificados são pacotes de informações; em termos de segurança, as chaves são a parte mais importante desse pacote. Certificados públicos contêm chaves públicas dos usuários, e certificados privados contêm chaves privadas dos usuários. Certificados públicos e privados estão vinculados. Certificados são únicos e não podem ser re-gerados. Note que se um certificado for perdido, ele não pode ser recuperado, então atualizar quaisquer aplicativos assinados com aquele certificado se torna impossível.
Criadores de aplicativos podem reutilizar um par de chaves pública/privada existente que está em um KeyStore disponível ou gerar um novo par.
No Android SDK, um novo par de chaves é gerado com o comando `keytool`. O seguinte comando cria um par de chaves RSA com um comprimento de chave de 2048 bits e um tempo de expiração de 7300 dias = 20 anos. O par de chaves gerado é armazenado no arquivo 'myKeyStore.jks', que está no diretório atual:

```bash
keytool -genkey -alias myDomain -keyalg RSA -keysize 2048 -validity 7300 -keystore myKeyStore.jks -storepass myStrongPassword
```

Armazenar com segurança sua chave secreta e garantir que ela permaneça secreta durante todo o seu ciclo de vida é de suma importância. Qualquer pessoa que ganhar acesso à chave será capaz de publicar atualizações para seus aplicativos com conteúdo que você não controla (adicionando assim recursos inseguros ou acessando conteúdo compartilhado com permissões baseadas em assinatura). A confiança que um usuário deposita em um aplicativo e seus desenvolvedores é baseada totalmente em tais certificados; proteção de certificado e gerenciamento seguro são, portanto, vitais para reputação e retenção de clientes, e chaves secretas nunca devem ser compartilhadas com outras pessoas. As chaves são armazenadas em um arquivo binário que pode ser protegido com uma senha; tais arquivos são referidos como _KeyStores_. Senhas do KeyStore devem ser fortes e conhecidas apenas pelo criador da chave. Por esta razão, as chaves são geralmente armazenadas em uma máquina de build dedicada à qual os desenvolvedores têm acesso limitado.
Um certificado Android deve ter um período de validade que seja mais longo que o do aplicativo associado (incluindo versões atualizadas do aplicativo). Por exemplo, o Google Play exigirá que os certificados permaneçam válidos até pelo menos 22 de outubro de 2033.

#### Assinando um Aplicativo

O objetivo do processo de assinatura é associar o arquivo do aplicativo (.apk) com a chave pública do desenvolvedor. Para alcançar isso, o desenvolvedor calcula um hash do arquivo APK e o criptografa com sua própria chave privada. Terceiros podem então verificar a autenticidade do aplicativo (por exemplo, o fato de que o aplicativo realmente vem do usuário que afirma ser o originador) descriptografando o hash criptografado com a chave pública do autor e verificando que ele corresponde ao hash real do arquivo APK.

Muitos Ambientes de Desenvolvimento Integrado (IDE) integram o processo de assinatura de aplicativo para torná-lo mais fácil para o usuário. Esteja ciente de que alguns IDEs armazenam chaves privadas em texto claro em arquivos de configuração; verifique isso novamente no caso de outros serem capazes de acessar tais arquivos e remova as informações se necessário.
Aplicativos podem ser assinados a partir da linha de comando com a ferramenta 'apksigner' fornecida pelo Android SDK (API level 24 e superior). Ela está localizada em `[SDK-Path]/build-tools/[version]`. Para API 24.0.2 e abaixo, você pode usar 'jarsigner', que faz parte do Java JDK. Detalhes sobre todo o processo podem ser encontrados na documentação oficial do Android; no entanto, um exemplo é dado abaixo para ilustrar o ponto.

```bash
apksigner sign --out mySignedApp.apk --ks myKeyStore.jks myUnsignedApp.apk
```

Neste exemplo, um aplicativo não assinado ('myUnsignedApp.apk') será assinado com uma chave privada do KeyStore do desenvolvedor 'myKeyStore.jks' (localizado no diretório atual). O aplicativo se tornará um aplicativo assinado chamado 'mySignedApp.apk' e estará pronto para lançar para lojas.

##### Zipalign

A ferramenta `zipalign` deve sempre ser usada para alinhar o arquivo APK antes da distribuição. Esta ferramenta alinha todos os dados não comprimidos (como imagens, arquivos brutos e limites de 4 bytes) dentro do APK, o que ajuda a melhorar o gerenciamento de memória durante o tempo de execução do aplicativo.

> Zipalign deve ser usado antes que o arquivo APK seja assinado com apksigner.

### Processo de Publicação

Distribuir aplicativos de qualquer lugar (seu próprio site, qualquer loja, etc.) é possível porque o ecossistema Android é aberto. No entanto, o Google Play é a loja mais conhecida, confiável e popular, e o próprio Google a fornece. A Amazon Appstore é a loja padrão confiável para dispositivos Kindle. Se os usuários quiserem instalar aplicativos de terceiros de uma fonte não confiável, eles devem explicitamente permitir isso com suas configurações de segurança do dispositivo.

Aplicativos podem ser instalados em um dispositivo Android a partir de uma variedade de fontes: localmente via USB,
via a loja oficial de aplicativos do Google (Google Play Store) ou de lojas alternativas.

Enquanto outros fornecedores podem revisar e aprovar aplicativos antes que sejam realmente publicados, o Google simplesmente escaneará em busca de assinaturas de malware conhecidas; isso minimiza o tempo entre o início do processo de publicação e a disponibilidade pública do aplicativo.

Publicar um aplicativo é bastante direto; a operação principal é tornar o arquivo APK assinado disponível para download. No Google Play, a publicação começa com a criação de conta e é seguida pela entrega do aplicativo através de uma interface dedicada. Detalhes estão disponíveis na [documentação oficial do Android](https://play.google.com/console/about/guides/releasewithconfidence/ "Revise as listas de verificação para planejar seu lançamento").