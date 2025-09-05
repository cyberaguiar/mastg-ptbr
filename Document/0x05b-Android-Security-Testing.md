# Teste de Segurança Android

Neste capítulo, mergulharemos na configuração de um ambiente de teste de segurança e apresentaremos alguns processos e técnicas práticas para testar a segurança de aplicativos Android. Estes são os blocos de construção para os casos de teste do MASTG.

## Configuração de Teste Android

Você pode configurar um ambiente de teste totalmente funcional em quase qualquer máquina executando Windows, Linux ou macOS.

### Dispositivo Host

No mínimo, você precisará de @MASTG-TOOL-0007 (que vem com as ferramentas de plataforma @MASTG-TOOL-0006), um emulador e um aplicativo para gerenciar as várias versões do SDK e componentes do framework. O Android Studio também vem com um aplicativo Android Virtual Device (AVD) Manager para criar imagens de emulador. Certifique-se de que os pacotes mais recentes de [ferramentas SDK](https://developer.android.com/studio/releases/sdk-tools) e [ferramentas de plataforma](https://developer.android.com/studio/releases/platform-tools) estejam instalados em seu sistema.

Além disso, você pode querer completar sua configuração de host instalando o @MASTG-TOOL-0005 se estiver planejando trabalhar com aplicativos contendo bibliotecas nativas.

Às vezes pode ser útil exibir ou controlar dispositivos a partir do computador. Para alcançar isso, você pode usar @MASTG-TOOL-0024.

### Dispositivo de Teste

Para análise dinâmica, você precisará de um dispositivo Android para executar o aplicativo alvo. Em princípio, você pode testar sem um dispositivo Android real e usar apenas o emulador. No entanto, aplicativos executam bastante lentamente em um emulador, e simuladores podem não dar resultados realistas. Testar em um dispositivo real torna o processo mais suave e o ambiente mais realista. Por outro lado, emuladores permitem que você altere facilmente versões do SDK ou crie múltiplos dispositivos. Uma visão geral completa dos prós e contras de cada abordagem está listada na tabela abaixo.

| Propriedade | Físico | Emulador/Simulador |
|---|---|---|
| Capacidade de restaurar | Softbricks são sempre possíveis, mas novo firmware normalmente ainda pode ser flashado. Hardbricks são muito raros. | Emuladores podem crashar ou se corromper, mas um novo pode ser criado ou um snapshot pode ser restaurado. |
| Reset | Pode ser restaurado para configurações de fábrica ou reflashado. | Emuladores podem ser deletados e recriados. |
| Snapshots | Não possível. | Suportado, ótimo para análise de malware. |
| Velocidade | Muito mais rápido que emuladores. | Tipicamente lento, mas melhorias estão sendo feitas. |
| Custo | Tipicamente começam em $200 para um dispositivo utilizável. Você pode exigir dispositivos diferentes, como um com ou sem sensor biométrico. | Tanto soluções gratuitas quanto comerciais existem. |
| Facilidade de rooting | Altamente dependente do dispositivo. | Tipicamente rooted por padrão. |
| Facilidade de detecção de emulador | Não é um emulador, então verificações de emulador não são aplicáveis. | Muitos artefatos existirão, tornando fácil detectar que o aplicativo está sendo executado em um emulador. |
| Facilidade de detecção de root | Mais fácil esconder root, já que muitos algoritmos de detecção de root verificam propriedades de emulador. Com Magisk Systemless root é quase impossível detectar. | Emuladores quase sempre acionarão algoritmos de detecção de root devido ao fato de serem construídos para teste com muitos artefatos que podem ser encontrados. |
| Interação de hardware | Interação fácil através de Bluetooth, NFC, 4G, Wi-Fi, biometria, câmera, GPS, giroscópio, ... | Geralmente bastante limitada, com entrada de hardware emulada (por exemplo, coordenadas GPS aleatórias) |
| Suporte a nível de API | Depende do dispositivo e da comunidade. Comunidades ativas continuarão distribuindo versões atualizadas (por exemplo, LineageOS), enquanto dispositivos menos populares podem receber apenas algumas atualizações. Alternar entre versões requer flashar o dispositivo, um processo tedioso. | Sempre suporta as versões mais recentes, incluindo lançamentos beta. Emuladores contendo níveis de API específicos podem facilmente ser baixados e lançados. |
| Suporte a biblioteca nativa | Bibliotecas nativas são geralmente construídas para dispositivos ARM, então funcionarão em um dispositivo físico. | Alguns emuladores executam em CPUs x86, então podem não ser capazes de executar bibliotecas nativas empacotadas. |
| Perigo de malware | Amostras de malware podem infectar um dispositivo, mas se você puder limpar o armazenamento do dispositivo e flashar um firmware limpo, restaurando assim para configurações de fábrica, isso não deve ser um problema. Esteja ciente de que há amostras de malware que tentam explorar a ponte USB. | Amostras de malware podem infectar um emulador, mas o emulador pode simplesmente ser removido e recriado. Também é possível criar snapshots e comparar diferentes snapshots para ajudar na análise de malware. Esteja ciente de que há provas de conceito de malware que tentam atacar o hypervisor. |

#### Testando em um Dispositivo Real

Quase qualquer dispositivo físico pode ser usado para teste, mas há algumas considerações a serem feitas. Primeiro, o dispositivo precisa ser rootable. Isso é tipicamente feito através de um exploit, ou através de um bootloader desbloqueado. Exploits nem sempre estão disponíveis, e o bootloader pode estar bloqueado permanentemente, ou pode ser desbloqueado apenas uma vez que o contrato da operadora tenha sido encerrado.

Os melhores candidatos são dispositivos flagship Google Pixel construídos para desenvolvedores. Esses dispositivos tipicamente vêm com um bootloader desbloqueável, firmware opensource, kernel, rádio disponível online e código fonte do OS oficial. As comunidades de desenvolvedores preferem dispositivos Google pois o OS é o mais próximo do projeto Android open source. Esses dispositivos geralmente têm as janelas de suporte mais longas com 2 anos de atualizações de OS e 1 ano de atualizações de segurança depois disso.

Alternativamente, o projeto [Android One](https://www.android.com/one/ "Android One") do Google contém dispositivos que receberão as mesmas janelas de suporte (2 anos de atualizações de OS, 1 ano de atualizações de segurança) e têm experiências quase stock. Embora tenha sido originalmente iniciado como um projeto para dispositivos de baixo custo, o programa evoluiu para incluir smartphones de médio e alto alcance, muitos dos quais são ativamente suportados pela comunidade de modding.

Dispositivos que são suportados pelo projeto [LineageOS](https://lineageos.org/ "LineageOS") também são muito bons candidatos para dispositivos de teste. Eles têm uma comunidade ativa, instruções de flashing e rooting fáceis de seguir e as versões mais recentes do Android são tipicamente rapidamente disponíveis como uma instalação Lineage. O LineageOS também continua o suporte para novas versões do Android muito depois que a OEM parou de distribuir atualizações.

Ao trabalhar com um dispositivo físico Android, você vai querer habilitar o Modo de Desenvolvedor e a depuração USB no dispositivo para usar a interface de depuração @MASTG-TOOL-0004. Desde o Android 4.2 (API level 16), o submenu **Opções do desenvolvedor** no aplicativo Configurações está oculto por padrão. Para ativá-lo, toque na seção **Número da compilação** da visualização **Sobre o telefone** sete vezes. Observe que a localização do campo do número da compilação varia ligeiramente por dispositivo. Por exemplo, em telefones LG, está em **Sobre o telefone** -> **Informações do software**. Uma vez que você tenha feito isso, **Opções do desenvolvedor** será mostrado na parte inferior do menu Configurações. Uma vez que as opções do desenvolvedor estejam ativadas, você pode habilitar a depuração com o interruptor **Depuração USB**.

#### Testando em um Emulador

Múltiplos emuladores existem, mais uma vez com seus próprios pontos fortes e fracos:

Emuladores gratuitos:

- [Android Virtual Device (AVD)](https://developer.android.com/studio/run/managing-avds.html "Criar e Gerenciar Dispositivos Virtuais") - O emulador android oficial, distribuído com o Android Studio.
- [Android X86](https://www.android-x86.org/ "Android X86") - Uma porta x86 da base de código Android

Emuladores comerciais:

- [Genymotion](https://www.genymotion.com/download/ "Genymotion") - Emulador maduro com muitos recursos, tanto como solução local quanto baseada em nuvem. Versão gratuita disponível para uso não comercial.
- [Corellium](https://corellium.com/ "Corellium") - Oferece virtualização de dispositivo personalizada através de uma solução baseada em nuvem ou on-prem.

Embora existam vários emuladores Android gratuitos, recomendamos usar o AVD, pois ele fornece recursos aprimorados apropriados para testar seu aplicativo em comparação com os outros. No restante deste guia, usaremos o AVD oficial para realizar testes.

O AVD suporta alguma emulação de hardware, como GPS ou SMS através de seus chamados [Controles Estendidos](https://developer.android.com/studio/run/advanced-emulator-usage#extended "Controles Estendidos"), bem como [sensores de movimento](https://developer.android.com/guide/topics/sensors/sensors_overview#test-with-the-android-emulator "Testando sensores de movimento em emuladores").

Você pode iniciar um Android Virtual Device (AVD) usando o AVD Manager no Android Studio ou iniciar o gerenciador AVD a partir da linha de comando com o comando `android`, que é encontrado no diretório de ferramentas do Android SDK:

```bash
./android avd
```

Várias ferramentas e VMs que podem ser usadas para testar um aplicativo dentro de um ambiente de emulador estão disponíveis:

- @MASTG-TOOL-0035
- [Nathan](https://github.com/mseclab/nathan "Nathan") (não atualizado desde 2016)

#### Obtendo Acesso Privilegiado

_Rooting_ (ou seja, modificando o OS para que você possa executar comandos como o usuário root) é recomendado para teste em um dispositivo real. Isso dá a você controle total sobre o sistema operacional e permite que você contorne restrições como o sandboxing de aplicativos. Esses privilégios por sua vez permitem que você use técnicas como injeção de código e function hooking mais facilmente.

Note que rooting é arriscado, e três consequências principais precisam ser esclarecidas antes de você prosseguir. Rooting pode ter os seguintes efeitos negativos:

- anulando a garantia do dispositivo (sempre verifique a política do fabricante antes de tomar qualquer ação)
- "bricking" o dispositivo, ou seja, tornando-o inoperante e inutilizável
- criando riscos de segurança adicionais (porque mitigações de exploit integradas são frequentemente removidas)

Você não deve rootear um dispositivo pessoal no qual você armazena suas informações privadas. Recomendamos obter um dispositivo de teste dedicado e barato. Muitos dispositivos mais antigos, como a série Nexus do Google, podem executar as versões mais recentes do Android e são perfeitamente adequados para teste.

**Você precisa entender que rootear seu dispositivo é, em última análise, SUA decisão e que a OWASP não deve de forma alguma ser responsabilizada por qualquer dano. Se você estiver incerto, busque aconselhamento especializado antes de iniciar o processo de rooting.**

##### Quais Móveis Podem Ser Roteados

Virtualmente qualquer móvel Android pode ser roteado. Versões comerciais do Android OS (que são evoluções do OS Linux no nível do kernel) são otimizadas para o mundo móvel. Alguns recursos foram removidos ou desabilitados para essas versões, por exemplo, a capacidade de usuários não privilegiados se tornarem o usuário 'root' (que tem privilégios elevados). Rootear um telefone significa permitir que usuários se tornem o usuário root, por exemplo, adicionando um executável Linux padrão chamado `su`, que é usado para mudar para outra conta de usuário.

Para rootear um dispositivo móvel, primeiro desbloqueie seu boot loader. O procedimento de desbloqueio depende do fabricante do dispositivo. No entanto, por razões práticas, rootear alguns dispositivos móveis é mais popular do que outros, particularmente quando se trata de teste de segurança: dispositivos criados pelo Google e fabricados por empresas como Samsung, LG e Motorola estão entre os mais populares, particularmente porque são usados por muitos desenvolvedores. A garantia do dispositivo não é anulada quando o boot loader é desbloqueado e o Google fornece muitas ferramentas para suportar o root em si.

##### Rooting com Magisk

Magisk ("Magic Mask") é uma maneira de rootear seu dispositivo Android. Sua especialidade está na maneira como as modificações no sistema são realizadas. Enquanto outras ferramentas de rooting alteram os dados reais na partição do sistema, o Magisk não (o que é chamado de "systemless"). Isso permite uma maneira de esconder as modificações de aplicativos sensíveis a root (por exemplo, para bancos ou jogos) e permite usar as atualizações OTA oficiais do Android sem a necessidade de desfazer o root do dispositivo antecipadamente.

Você pode se familiarizar com o Magisk lendo a [documentação oficial no GitHub](https://topjohnwu.github.io/Magisk/ "Documentação do Magisk"). Se você não tem o Magisk instalado, pode encontrar instruções de instalação na [documentação](https://topjohnwu.github.io/Magisk/ "Documentação do Magisk"). Se você usa uma versão oficial do Android e planeja atualizá-la, o Magisk fornece um [tutorial no GitHub](https://topjohnwu.github.io/Magisk/ota.html "Instalação OTA").

Além disso, desenvolvedores podem usar o poder do Magisk para criar módulos personalizados e [submetê-los](https://github.com/Magisk-Modules-Repo/submission "Submissão") ao repositório oficial de [Módulos Magisk](https://github.com/Magisk-Modules-Repo "Magisk-Modules-Repo"). Módulos submetidos podem então ser instalados dentro do aplicativo Magisk Manager. Um desses módulos instaláveis é uma versão systemless do @MASTG-TOOL-0027 (disponível para versões SDK até 27).

##### Detecção de Root

Uma lista extensiva de métodos de detecção de root é apresentada no capítulo "Testando Defesas Anti-Reversão no Android".

Para uma construção típica de segurança de aplicativo móvel, você geralmente vai querer testar uma construção de depuração com detecção de root desabilitada. Se tal construção não estiver disponível para teste, você pode desabilitar a detecção de root de uma variedade de maneiras que serão introduzidas mais tarde neste livro.