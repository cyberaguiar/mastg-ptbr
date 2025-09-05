# Taxonomia de Aplicativos Móveis

Quando usamos o termo "aplicativo móvel" ou "app móvel", estamos nos referindo a um programa de computador autocontido projetado para executar em um dispositivo móvel. No momento da publicação, os sistemas operacionais Android e iOS compreendem coletivamente [mais de 99% da participação de mercado de SO móvel](https://www.idc.com/promo/smartphone-market-share/os) e o uso da Internet móvel superou em muito o uso da Internet desktop. Isso significa que aplicativos móveis são os [tipos mais difundidos de aplicativos com capacidade para Internet](https://www.idc.com/promo/smartphone-market-share/os).

Além disso, este guia usa o termo "app" como um termo geral que se refere a qualquer tipo de aplicação que executa em um SO móvel. Normalmente, apps executam diretamente na plataforma para a qual são projetados, executam em cima do navegador móvel do dispositivo inteligente, ou usam uma mistura desses dois métodos.

Neste capítulo, discutiremos os seguintes tipos de apps:

- [Apps Nativos](#apps-nativos)
- [Frameworks Móveis Multiplataforma](#frameworks-móveis-multiplataforma)
- [Apps Web](#apps-web)
- [Apps Híbridos](#apps-híbridos)
- [Progressive Web Apps](#progressive-web-apps)

## Apps Nativos

Se um aplicativo móvel é desenvolvido com um Software Development Kit (SDK) para desenvolver apps específicos para um SO móvel, eles são referidos como _nativos_ ao seu SO. Se estamos discutindo um app nativo, presumimos que foi implementado em uma linguagem de programação padrão para aquele sistema operacional móvel - Objective-C ou Swift para iOS, e Java ou Kotlin para Android.

Porque são projetados para um SO específico com as ferramentas destinadas a esse SO, _apps nativos_ têm a capacidade de fornecer o desempenho mais rápido com o maior grau de confiabilidade. Eles geralmente aderem a princípios de design específicos da plataforma (ex: [Android Design Principles](https://developer.android.com/design "Android Design Principles")), o que geralmente leva a uma interface de usuário (UI) mais consistente em comparação com apps _híbridos_ ou _web_. Devido à sua integração próxima com o sistema operacional, _apps nativos_ geralmente podem acessar diretamente quase todos os componentes do dispositivo (câmera, sensores, armazenamentos de chaves com suporte de hardware, etc.).

No entanto, como o Android fornece dois kits de desenvolvimento - o Android SDK e o Android NDK, há alguma ambiguidade no termo _apps nativos_ para esta plataforma. Enquanto o SDK (baseado nas linguagens de programação Java e Kotlin) é o padrão para desenvolver apps, o NDK da plataforma (ou Native Development Kit) é um kit C/C++ usado para desenvolver bibliotecas binárias que podem acessar diretamente APIs de nível inferior (como OpenGL). Essas bibliotecas podem ser incluídas em apps regulares construídos com o SDK. Portanto, dizemos que _apps nativos_ Android (ou seja, construídos com o SDK) podem ter código _nativo_ construído com o NDK.

## Frameworks Móveis Multiplataforma

A desvantagem mais óbvia dos _apps nativos_ é que eles são limitados a uma plataforma específica. Se desenvolvedores querem construir seu app para Android e iOS, é preciso manter duas bases de código independentes, ou introduzir ferramentas de desenvolvimento frequentemente complexas para portar uma única base de código para duas plataformas.

Aqui estão alguns frameworks móveis multiplataforma que permitem desenvolvedores compilar uma única base de código para diferentes alvos, incluindo Android e iOS:

- [Xamarin](https://dotnet.microsoft.com/apps/xamarin "Xamarin")
- [MAUI](https://dotnet.microsoft.com/en-us/apps/maui ".NET MAUI")
- [Flutter](https://flutter.dev/ "Google Flutter")
- [React Native](https://reactnative.dev/ "React Native")
- [Unity](https://unity.com/ "Unity")

Se um app é desenvolvido usando esses frameworks, o app usará as APIs internas nativas de cada sistema e oferecerá desempenho equivalente a apps nativos. Além disso, esses apps podem fazer uso de todas as capacidades do dispositivo, incluindo GPS, acelerômetro, câmera, sistema de notificação, etc. Mesmo que um app criado usando um desses frameworks seja funcionalmente equivalente a um app nativo verdadeiro, eles normalmente não são referidos como tal. O termo _app nativo_ é usado para apps criados com o SDK nativo do SO, enquanto apps criados usando um desses frameworks são tipicamente chamados de apps multiplataforma.

É importante saber quando um app usa um framework móvel multiplataforma, porque eles tipicamente requerem ferramentas específicas para realizar análise estática ou dinâmica. A lógica real da aplicação está tipicamente localizada em arquivos específicos do framework dentro do app, mesmo que o app também contenha o código típico que você veria em um _app nativo_. Este código nativo é no entanto geralmente usado apenas para inicializar o framework multiplataforma e fornecer ligações entre a API do sistema nativo e o SDK do framework através das chamadas ligações específicas da plataforma.

Embora seja raro, apps podem combinar código nativo e frameworks multiplataforma, ou até mesmo múltiplos frameworks multiplataforma, então é importante identificar todas as tecnologias usadas para cobrir corretamente toda a superfície de ataque do app.

## Apps Web

Apps web móveis (ou simplesmente, _apps web_) são sites projetados para parecer e sentir como um _app nativo_. Esses apps executam no navegador do dispositivo e são geralmente desenvolvidos em HTML5, muito como uma página web moderna. Ícones de inicializador podem ser usados para paralelizar a mesma sensação de acessar um _app nativo_; no entanto, esses ícones são essencialmente os mesmos que um favorito de navegador, simplesmente abrindo o navegador web padrão para carregar a página web referenciada.

Porque executam dentro dos confins de um navegador, apps web têm integração limitada com os componentes gerais do dispositivo (ou seja, eles estão "sandboxed") e seu desempenho é geralmente inferior comparado a apps nativos. Como desenvolvedores geralmente visam múltiplas plataformas com um app web, suas UIs geralmente não seguem os princípios de design de nenhuma plataforma específica. No entanto, _apps web_ são populares porque desenvolvedores podem usar uma única base de código para reduzir custos de desenvolvimento e manutenção e distribuir atualizações sem passar pelas lojas de apps específicas da plataforma. Por exemplo, uma mudança no arquivo HTML para um _app web_ pode servir como uma atualização viável e multiplataforma, enquanto uma atualização para um app baseado em loja requer consideravelmente mais esforço.

## Apps Híbridos

_Apps híbridos_ são um tipo específico de _app multiplataforma_ que tenta se beneficiar dos melhores aspectos de _apps nativos_ e _web apps_. Este tipo de app executa como um _app nativo_, mas a maioria dos processos depende de tecnologias web, significando que uma porção do app executa em um navegador web embutido (comumente chamado "WebView"). Como tal, _apps híbridos_ herdam tanto prós quanto contras de _apps nativos_ e _web apps_. Esses apps podem usar uma camada de abstração web-to-native para acessar capacidades do dispositivo que não são acessíveis a um _app web_ puro. Dependendo do framework usado para desenvolvimento, uma base de código de _app híbrido_ pode gerar múltiplos apps que visam diferentes plataformas e aproveitam elementos de UI que se assemelham de perto à plataforma original do dispositivo.

Aqui estão alguns frameworks populares para desenvolver _apps híbridos_:

- [Apache Cordova](https://cordova.apache.org/ "Apache Cordova")
- [Framework 7](https://framework7.io/ "Framework 7")
- [Ionic](https://ionicframework.com/ "Ionic")
- [Native Script](https://www.nativescript.org/ "Native Script")
- [Onsen UI](https://onsen.io/ "Onsen UI")
- [Sencha Ext JS](https://www.sencha.com/products/extjs/ "Sencha Ext JS")

## Progressive Web Apps

_Progressive web apps_ (PWAs) combinam diferentes padrões abertos da web oferecidos por navegadores modernos para fornecer benefícios de uma experiência móvel rica. Um Web App Manifest, que é um arquivo JSON simples, pode ser usado para configurar o comportamento do app após "instalação". Esses apps carregam como páginas web regulares, mas diferem de apps web usuais de várias maneiras.

Por exemplo, é possível trabalhar offline e o acesso ao hardware do dispositivo móvel é possível, o que tem sido uma capacidade que estava disponível apenas para _apps nativos_. PWAs são suportados por Android e iOS, mas nem todos os recursos de hardware estão disponíveis ainda. Por exemplo, Push Notifications, Face ID no iPhone X, ou ARKit para realidade aumentada não estão disponíveis ainda no iOS.