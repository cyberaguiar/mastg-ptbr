# Teste de Segurança iOS

Neste capítulo, vamos mergulhar na configuração de um ambiente de teste de segurança e apresentar alguns processos e técnicas práticas para testar a segurança de aplicativos iOS. Estes são os blocos de construção para os casos de teste MASTG.

## Configuração de Teste iOS

Embora você possa usar um computador host Linux ou Windows para testes, descobrirá que muitas tarefas são difíceis ou impossíveis nessas plataformas. Além disso, o ambiente de desenvolvimento Xcode e o iOS SDK estão disponíveis apenas para macOS. Isso significa que você definitivamente vai querer trabalhar no macOS para análise de código fonte e depuração (também facilita o teste de caixa preta).

### Dispositivo Host

A seguir está a configuração mais básica para teste de aplicativos iOS:

- Idealmente computador host macOS com direitos de administrador
- @MASTG-TOOL-0070 e @MASTG-TOOL-0071 instalados.
- Rede Wi-Fi que permite tráfego cliente-a-cliente.
- Pelo menos um dispositivo iOS jailbroken (da versão iOS desejada).
- @MASTG-TOOL-0097 ou outra ferramenta de proxy de interceptação.

### Obtendo o UDID de um dispositivo iOS

O UDID é uma sequência única de 40 dígitos de letras e números para identificar um dispositivo iOS. Você pode encontrar o UDID do seu dispositivo iOS no macOS Catalina em diante no aplicativo Finder, já que o iTunes não está mais disponível no Catalina. Abra o Finder e selecione o dispositivo iOS conectado na barra lateral.

<img src="Images/Chapters/0x06b/finder_ipad_view.png" width="100%" />

Clique no texto contendo o modelo, capacidade de armazenamento e informações da bateria, e ele exibirá o número de série, UDID e modelo:

<img src="Images/Chapters/0x06b/finder_unveil_udid.png" width="100%" />

Você pode copiar o UDID clicando com o botão direito nele.

Também é possível obter o UDID via várias ferramentas de linha de comando no macOS enquanto o dispositivo está conectado via USB:

- Usando a ferramenta [I/O Registry Explorer](https://developer.apple.com/library/archive/documentation/DeviceDrivers/Conceptual/IOKitFundamentals/TheRegistry/TheRegistry.html "I/O Registry Explorer") `ioreg`:

    ```sh
    $ ioreg -p IOUSB -l | grep "USB Serial"
    |         "USB Serial Number" = "9e8ada44246cee813e2f8c1407520bf2f84849ec"
    ```

- Usando @MASTG-TOOL-0126:

    ```sh
    $ idevice_id -l
    316f01bd160932d2bf2f95f1f142bc29b1c62dbc
    ```

- Usando o system_profiler:

    ```sh
    $ system_profiler SPUSBDataType | sed -n -e '/iPad/,/Serial/p;/iPhone/,/Serial/p;/iPod/,/Serial/p' | grep "Serial Number:"
    2019-09-08 10:18:03.920 system_profiler[13251:1050356] SPUSBDevice: IOCreatePlugInInterfaceForService failed 0xe00002be
                Serial Number: 64655621de6ef5e56a874d63f1e1bdd14f7103b1
    ```

- Usando instruments:

    ```sh
    instruments -s devices
    ```

### Testando em um dispositivo real (Jailbroken)

Você deve ter um iPhone ou iPad jailbroken para executar testes. Esses dispositivos permitem acesso root e instalação de ferramentas, tornando o processo de teste de segurança mais direto. Se você não tem acesso a um dispositivo jailbroken, pode aplicar as soluções alternativas descritas posteriormente neste capítulo, mas esteja preparado para uma experiência mais difícil.

### Testando no iOS Simulator

Ao contrário do emulador Android, que emula completamente o hardware de um dispositivo Android real, o simulador iOS SDK oferece uma _simulação_ de nível superior de um dispositivo iOS. Mais importante, os binários do emulador são compilados para código x86 em vez de código ARM. Aplicativos compilados para um dispositivo real não são executados, tornando o simulador inútil para análise de caixa preta e engenharia reversa.

### Testando em um Emulador

@MASTG-TOOL-0108 é o único emulador iOS disponível publicamente. É uma solução SaaS empresarial com um modelo de licença por usuário e não oferece licenças comunitárias.

### Obtendo Acesso Privilegiado

O jailbreaking do iOS é frequentemente comparado ao rooting do Android, mas o processo é realmente bastante diferente. Para explicar a diferença, primeiro revisaremos os conceitos de "rooting" e "flashing" no Android.

- **Rooting**: Isso normalmente envolve instalar o binário `su` no sistema ou substituir todo o sistema por uma ROM personalizada com root. Exploits não são necessários para obter acesso root, desde que o bootloader esteja acessível.
- **Flashing de ROMs personalizadas**: Isso permite substituir o OS que está sendo executado no dispositivo após desbloquear o bootloader. O bootloader pode exigir um exploit para desbloqueá-lo.

Em dispositivos iOS, flashing de ROMs personalizadas é impossível porque o bootloader iOS só permite inicializar e flashear imagens assinadas pela Apple. É por isso que mesmo imagens iOS oficiais não podem ser instaladas se não forem assinadas pela Apple, e torna os downgrades do iOS possíveis apenas enquanto a versão anterior do iOS ainda estiver sendo assinada.

O propósito do jailbreaking é desativar as proteções do iOS (mecanismos de code signing da Apple em particular) para que código arbitrário não assinado possa ser executado no dispositivo (por exemplo, código personalizado ou baixado de lojas de aplicativos alternativas como @MASTG-TOOL-0047 ou @MASTG-TOOL-0064). A palavra "jailbreak" é uma referência coloquial a ferramentas all-in-one que automatizam o processo de desativação.

Desenvolver um jailbreak para uma determinada versão do iOS não é fácil. Como testador de segurança, você provavelmente vai querer usar ferramentas de jailbreak disponíveis publicamente. Ainda assim, recomendamos estudar as técnicas que foram usadas para fazer jailbreak em várias versões do iOS - você encontrará muitos exploits interessantes e aprenderá muito sobre os internos do OS. Por exemplo, Pangu9 para iOS 9.x [explorou pelo menos cinco vulnerabilidades](https://www.theiphonewiki.com/wiki/Jailbreak_Exploits "Jailbreak Exploits"), incluindo um bug de kernel use-after-free (CVE-2015-6794) e uma vulnerabilidade de acesso arbitrário ao sistema de arquivos no aplicativo Fotos (CVE-2015-7037).

Alguns aplicativos tentam detectar se o dispositivo iOS no qual estão sendo executados está com jailbreak. Isso ocorre porque o jailbreak desativa alguns dos mecanismos de segurança padrão do iOS. No entanto, existem várias maneiras de contornar essas detecções, e as apresentaremos no capítulo ["Defesas Anti-Reversão iOS"](0x06j-Testing-Resiliency-Against-Reverse-Engineering.md).

#### Benefícios do Jailbreaking

Usuários finais frequentemente fazem jailbreak em seus dispositivos para ajustar a aparência do sistema iOS, adicionar novos recursos e instalar aplicativos de terceiros de lojas de aplicativos não oficiais. Para um testador de segurança, no entanto, fazer jailbreak em um dispositivo iOS tem ainda mais benefícios. Eles incluem, mas não se limitam a:

- Acesso root ao sistema de arquivos.
- Possibilidade de executar aplicativos que não foram assinados pela Apple (o que inclui muitas ferramentas de segurança).
- Depuração e análise dinâmica irrestritas.
- Acesso ao runtime Objective-C ou Swift.

#### Tipos de Jailbreak

Existem jailbreaks _tethered_, _semi-tethered_, _semi-untethered_ e _untethered_.

- Jailbreaks tethered não persistem através de reinicializações, portanto, reaplicar jailbreaks requer que o dispositivo esteja conectado (tethered) a um computador durante cada reinicialização. O dispositivo pode nem reinicializar se o computador não estiver conectado.

- Jailbreaks semi-tethered não podem ser reaplicados a menos que o dispositivo esteja conectado a um computador durante a reinicialização. O dispositivo também pode inicializar no modo não jailbroken por conta própria.

- Jailbreaks semi-untethered permitem que o dispositivo inicialize por conta própria, mas os patches do kernel (ou modificações user-land) para desativar o code signing não são aplicados automaticamente. O usuário deve refazer o jailbreak do dispositivo iniciando um aplicativo ou visitando um site (não requerendo conexão com um computador, daí o termo untethered).

- Jailbreaks untethered são a escolha mais popular para usuários finais porque precisam ser aplicados apenas uma vez, após o que o dispositivo estará permanentemente com jailbreak.

#### Advertências e Considerações

Desenvolver um jailbreak para iOS está se tornando cada vez mais complicado à medida que a Apple continua a endurecer seu OS. Sempre que a Apple toma conhecimento de uma vulnerabilidade, ela é corrigida e uma atualização do sistema é enviada para todos os usuários. Como não é possível fazer downgrade para uma versão específica do iOS, e como a Apple só permite que você atualize para a versão mais recente do iOS, é um desafio ter um dispositivo que esteja executando uma versão do iOS para a qual um jailbreak esteja disponível. Algumas vulnerabilidades não podem ser corrigidas por software, como o exploit [checkm8](https://www.theiphonewiki.com/wiki/Checkm8_Exploit "Checkm8 exploit") afetando o BootROM de todas as CPUs até A12.

Se você tem um dispositivo com jailbreak que usa para teste de segurança, mantenha-o como está, a menos que tenha 100% de certeza de que pode refazer o jailbreak após atualizar para a versão mais recente do iOS. Considere obter um (ou vários) dispositivo(s) reserva(s) (que serão atualizados com cada lançamento principal do iOS) e aguardar até que um jailbreak seja lançado publicamente. A Apple geralmente é rápida em lançar um patch assim que um jailbreak é lançado publicamente, então você só tem alguns dias para fazer downgrade (se ainda estiver sendo assinado pela Apple) para a versão do iOS afetada e aplicar o jailbreak.

As atualizações do iOS são baseadas em um processo challenge-response (gerando os chamados SHSH blobs como resultado). O dispositivo permitirá a instalação do OS apenas se a resposta ao desafio for assinada pela Apple. Isso é o que os pesquisadores chamam de "janela de assinatura", e é a razão pela qual você não pode simplesmente armazenar o pacote de firmware OTA que baixou e carregá-lo no dispositivo sempre que quiser. Durante atualizações menores do iOS, duas versões podem estar sendo assinadas pela Apple (a mais recente e a versão anterior do iOS). Esta é a única situação em que você pode fazer downgrade do dispositivo iOS. Você pode verificar a janela de assinatura atual e baixar o firmware OTA do site [IPSW Downloads](https://ipsw.me "IPSW Downloads").

Para alguns dispositivos e versões do iOS, é possível fazer downgrade para versões mais antigas caso os SHSH blobs para esse dispositivo tenham sido coletados quando a janela de assinatura estava ativa. Mais informações sobre isso podem ser encontradas no [cfw iOS Guide - Saving Blobs](https://ios.cfw.guide/saving-blobs/)

#### Qual Ferramenta de Jailbreak Usar

Diferentes versões do iOS requerem diferentes técnicas de jailbreak. [Determine se um jailbreak público está disponível para sua versão do iOS](https://appledb.dev/ "Apple DB"). Cuidado com ferramentas falsas e spyware, que frequentemente se escondem atrás de nomes de domínio semelhantes ao nome do grupo/autor do jailbreak.

A cena de jailbreak do iOS evolui tão rapidamente que fornecer instruções atualizadas é difícil. No entanto, podemos indicar algumas fontes atualmente confiáveis.

- [AppleDB](https://appledb.dev/ "AppleDB")
- [The iPhone Wiki](https://www.theiphonewiki.com/ "The iPhone Wiki")
- [Redmond Pie](https://www.redmondpie.com/ "Redmone Pie")
- [Reddit Jailbreak](https://www.reddit.com/r/jailbreak/ "Reddit Jailbreak")

> Observe que qualquer modificação que você fizer em seu dispositivo é por sua conta e risco. Embora o jailbreak seja tipicamente seguro, as coisas podem dar errado e você pode acabar brickando seu dispositivo. Nenhuma outra parte, exceto você mesmo, pode ser responsabilizada por qualquer dano.