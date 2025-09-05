# Taxonomia de Aplicativos Móveis

Quando usamos o termo "aplicativo móvel" ou "app móvel", estamos nos referindo a um programa de computador autocontido projetado para executar em um dispositivo móvel. No momento da publicação, os sistemas operacionais Android e iOS representam juntos [mais de 99% de participação no mercado de SOs móveis](https://www.idc.com/promo/smartphone-market-share/os) e o uso da Internet em dispositivos móveis já superou bastante o uso em desktops. Isso significa que apps móveis são [o tipo mais difundido de aplicativos com acesso à Internet](https://www.idc.com/promo/smartphone-market-share/os).

Neste guia, o termo "app" é usado de forma geral para se referir a qualquer tipo de aplicativo que roda em um SO móvel. Normalmente, os apps executam diretamente na plataforma para a qual foram projetados, rodam em um navegador móvel do dispositivo ou usam uma combinação desses dois métodos.

Neste capítulo, discutiremos os seguintes tipos de apps:

- [Native Apps](#native-apps)
- [Cross-platform Mobile Frameworks](#cross-platform-mobile-frameworks)
- [Web Apps](#web-apps)
- [Hybrid Apps](#hybrid-apps)
- [Progressive Web Apps](#progressive-web-apps)

## Native Apps

Se um app móvel é desenvolvido com um Software Development Kit (SDK) específico para um SO móvel, ele é considerado _nativo_ para aquele sistema. Ao discutirmos um app nativo, presumimos que ele foi implementado em uma linguagem de programação padrão para o sistema operacional móvel — Objective-C ou Swift para iOS, e Java ou Kotlin para Android.

Por serem projetados para um SO específico com as ferramentas destinadas a ele, _native apps_ têm capacidade de oferecer o desempenho mais rápido com o maior grau de confiabilidade. Eles geralmente seguem princípios de design específicos da plataforma (por exemplo, os [Android Design Principles](https://developer.android.com/design "Android Design Principles")), o que normalmente resulta em uma interface do usuário (UI) mais consistente em comparação com _hybrid_ ou _web apps_. Devido à integração estreita com o sistema operacional, _native apps_ geralmente podem acessar diretamente quase todos os componentes do dispositivo (câmera, sensores, armazenamentos de chaves baseados em hardware etc.).

No entanto, como o Android fornece dois kits de desenvolvimento — o Android SDK e o Android NDK — há alguma ambiguidade no termo _native apps_ para essa plataforma. Enquanto o SDK (baseado nas linguagens Java e Kotlin) é o padrão para desenvolvimento de apps, o NDK (Native Development Kit) é um kit em C/C++ usado para desenvolver bibliotecas binárias que podem acessar APIs de baixo nível diretamente (como OpenGL). Essas bibliotecas podem ser incluídas em apps regulares construídos com o SDK. Portanto, dizemos que _native apps_ Android (isto é, construídos com o SDK) podem conter código _nativo_ desenvolvido com o NDK.

## Cross-platform Mobile Frameworks

A desvantagem mais evidente dos _native apps_ é que eles ficam limitados a uma plataforma específica. Se desenvolvedores quiserem criar o app para Android e iOS, precisam manter dois códigos independentes ou introduzir ferramentas de desenvolvimento muitas vezes complexas para portar uma única base de código para duas plataformas.

A seguir estão alguns frameworks móveis multiplataforma que permitem compilar uma única base de código para diferentes alvos, incluindo Android e iOS:

- [Xamarin](https://dotnet.microsoft.com/apps/xamarin "Xamarin")
- [MAUI](https://dotnet.microsoft.com/en-us/apps/maui ".NET MAUI")
- [Flutter](https://flutter.dev/ "Google Flutter")
- [React Native](https://reactnative.dev/ "React Native")
- [Unity](https://unity.com/ "Unity")

Se um app é desenvolvido usando esses frameworks, ele usará as APIs internas nativas de cada sistema e oferecerá desempenho equivalente ao de _native apps_. Além disso, esses apps podem usar todos os recursos do dispositivo, incluindo GPS, acelerômetro, câmera, sistema de notificações etc. Mesmo que um app criado com um desses frameworks seja funcionalmente equivalente a um verdadeiro app nativo, normalmente não é referido como tal. O termo _native app_ é usado para apps criados com o SDK nativo do sistema operacional, enquanto apps criados com um desses frameworks são geralmente chamados de apps multiplataforma.

É importante saber quando um app usa um framework móvel multiplataforma, pois eles normalmente exigem ferramentas específicas para realizar análise estática ou dinâmica. A lógica real da aplicação geralmente está localizada em arquivos específicos do framework dentro do app, embora o app também contenha o código típico que você veria em um _native app_. Esse código nativo normalmente é usado apenas para inicializar o framework multiplataforma e fornecer ligações entre a API nativa do sistema e o SDK do framework por meio das chamadas platform-specific bindings.

Embora seja raro, apps podem combinar código nativo e frameworks multiplataforma ou até múltiplos frameworks ao mesmo tempo, portanto é importante identificar todas as tecnologias utilizadas para cobrir corretamente toda a superfície de ataque do app.

## Web Apps

Web apps móveis (ou simplesmente _web apps_) são sites projetados para parecer e se comportar como um _native app_. Esses apps rodam no navegador do dispositivo e normalmente são desenvolvidos em HTML5, como uma página web moderna. Ícones na tela inicial podem ser usados para simular a mesma sensação de acesso a um _native app_; no entanto, esses ícones são essencialmente o equivalente a um favorito do navegador, apenas abrindo o navegador padrão para carregar a página referenciada.

Como rodam dentro dos limites de um navegador, _web apps_ têm integração limitada com os componentes gerais do dispositivo (ou seja, são "sandboxed") e seu desempenho geralmente é inferior ao de _native apps_. Como desenvolvedores normalmente miram várias plataformas com um _web app_, suas UIs geralmente não seguem os princípios de design de uma plataforma específica. No entanto, _web apps_ são populares porque desenvolvedores podem usar uma única base de código para reduzir custos de desenvolvimento e manutenção e distribuir atualizações sem passar pelas lojas específicas de cada plataforma. Por exemplo, uma alteração em um arquivo HTML de um _web app_ pode servir como uma atualização viável e multiplataforma, enquanto uma atualização em um app distribuído em loja exige muito mais esforço.

## Hybrid Apps

_Hybrid apps_ são um tipo específico de _cross-platform app_ que busca aproveitar o melhor de _native_ e _web apps_. Esse tipo de app executa como um _native app_, mas a maior parte dos processos depende de tecnologias web, o que significa que parte do app roda em um navegador embutido (comumente chamado de "WebView"). Assim, _hybrid apps_ herdam tanto os prós quanto os contras de _native_ e _web apps_. Esses apps podem usar uma camada de abstração web-to-native para acessar recursos do dispositivo que não estão disponíveis para um _web app_ puro. Dependendo do framework usado no desenvolvimento, uma base de código _hybrid app_ pode gerar múltiplos apps que miram diferentes plataformas e aproveitar elementos de UI que se assemelham aos da plataforma original do dispositivo.

Aqui estão alguns frameworks populares para desenvolver _hybrid apps_:

- [Apache Cordova](https://cordova.apache.org/ "Apache Cordova")
- [Framework 7](https://framework7.io/ "Framework 7")
- [Ionic](https://ionicframework.com/ "Ionic")
- [Native Script](https://www.nativescript.org/ "Native Script")
- [Onsen UI](https://onsen.io/ "Onsen UI")
- [Sencha Ext JS](https://www.sencha.com/products/extjs/ "Sencha Ext JS")

## Progressive Web Apps

_Progressive Web Apps_ (PWAs) combinam diferentes padrões abertos da web oferecidos por navegadores modernos para fornecer os benefícios de uma experiência móvel rica. Um Web App Manifest, que é um arquivo JSON simples, pode ser usado para configurar o comportamento do app após a "instalação". Esses apps carregam como páginas web regulares, mas diferem de _web apps_ usuais em vários aspectos.

Por exemplo, é possível trabalhar offline e acessar o hardware do dispositivo móvel, algo que antes estava disponível apenas para _native apps_. PWAs são suportados tanto em Android quanto em iOS, mas nem todos os recursos de hardware estão disponíveis. Por exemplo, Push Notifications, Face ID no iPhone X ou ARKit para realidade aumentada ainda não estão disponíveis no iOS.
