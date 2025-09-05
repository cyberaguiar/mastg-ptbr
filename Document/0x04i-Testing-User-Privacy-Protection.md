
# Proteção de Privacidade do Usuário em Aplicativos Móveis

## Visão Geral

**ISENÇÃO DE RESPONSABILIDADE IMPORTANTE:** O MASTG não é um manual jurídico e não entrará nos detalhes específicos do GDPR ou de outra legislação possivelmente relevante aqui. Em vez disso, este capítulo apresentará os tópicos relacionados à proteção da privacidade do usuário, fornecerá referências essenciais para seus próprios esforços de pesquisa e dará testes ou diretrizes que determinam se um aplicativo adere aos requisitos relacionados à privacidade listados no OWASP MASVS.

### O Problema Principal

Aplicativos móveis lidam com todos os tipos de dados sensíveis do usuário, desde informações de identificação e bancárias até dados de saúde, então tanto os desenvolvedores quanto o público estão compreensivelmente preocupados com como esses dados são tratados e onde eles acabam. Também vale a pena discutir os "benefícios que os usuários obtêm ao usar os aplicativos" versus "o preço real que estão pagando por isso" (muitas vezes sem nem mesmo estar cientes disso).

### A Solução (pré-2020)

Para garantir que os usuários estejam adequadamente protegidos, legislações como o [Regulamento Geral de Proteção de Dados (GDPR)](https://gdpr-info.eu/ "GDPR") da União Europeia na Europa foram desenvolvidas e implantadas (aplicáveis desde 25 de maio de 2018). Essas leis podem forçar os desenvolvedores a serem mais transparentes em relação ao tratamento de dados sensíveis do usuário, o que geralmente é implementado com políticas de privacidade.

### O Desafio

Considere estas dimensões da privacidade de aplicativos móveis:

- **Conformidade do Desenvolvedor**: Os desenvolvedores precisam estar cientes das leis sobre privacidade do usuário para que seu trabalho seja compatível. Idealmente, os seguintes princípios devem ser seguidos:
    - Abordagem **Privacy-by-Design** (Art. 25 GDPR, "Proteção de dados desde a concepção e por padrão").
    - **Princípio do Privilégio Mínimo** ("Todo programa e todo usuário do sistema deve operar usando o menor conjunto de privilégios necessário para concluir o trabalho.")
- **Educação do Usuário**: Os usuários precisam ser educados sobre seus dados sensíveis e informados sobre como usar o aplicativo corretamente (para garantir o manuseio e processamento seguros de suas informações).

> Nota: Na maioria das vezes, os aplicativos afirmarão lidar com certos dados, mas na realidade esse não é o caso. O artigo IEEE ["Engineering Privacy in Smartphone Apps: A Technical Guideline Catalog for App Developers" de Majid Hatamian](https://drive.google.com/file/d/1cp7zrqJuVkftJ0DARNN40Ga_m_tEhIrQ/view?usp=sharing) dá uma introdução muito boa a este tópico.

### Metas para Proteção de Dados

Quando um aplicativo solicita informações pessoais de um usuário, o usuário precisa saber por que o aplicativo precisa desses dados e como eles são usados pelo aplicativo. Se houver um terceiro fazendo o processamento real dos dados, o aplicativo também deve informar o usuário sobre isso.

Assim como a tríade clássica de metas de proteção de segurança: confidencialidade, integridade e disponibilidade, existem três metas de proteção que foram propostas para proteção de dados:

- **Desvinculabilidade**:
    - Os dados relevantes para privacidade dos usuários são desvinculáveis de qualquer outro conjunto de dados relevantes para privacidade fora do domínio.
    - Inclui: minimização de dados, anonimização, pseudonimização, etc.
- **Transparência**:
    - Os usuários devem saber como solicitar todas as informações que o aplicativo tem sobre eles e estar cientes de todas as informações que o aplicativo tem sobre eles.
    - Inclui: políticas de privacidade, educação do usuário, mecanismos adequados de registro e auditoria, etc.
- **Intervenibilidade**:
    - Os usuários devem saber como corrigir suas informações pessoais, solicitar sua exclusão, retirar qualquer consentimento dado a qualquer momento e receber instruções sobre como fazê-lo.
    - Inclui: configurações de privacidade diretamente no aplicativo, pontos únicos de contato para solicitações de intervenção de indivíduos (por exemplo, chat no aplicativo, número de telefone, e-mail), etc.

> Para mais detalhes, consulte a Seção 5.1.1 "Introdução às metas de proteção de dados" no ["Privacidade e proteção de dados em aplicações móveis" da ENISA](https://www.enisa.europa.eu/publications/privacy-and-data-protection-in-mobile-applications "ENISA - Privacidade e proteção de dados em aplicações móveis").

Como é muito desafiador (se não impossível em muitos casos) abordar tanto as metas de proteção de segurança quanto de privacidade ao mesmo tempo, vale a pena examinar uma visualização na publicação IEEE [Metas de Proteção para Engenharia de Privacidade](https://ieeexplore.ieee.org/document/7163220) chamada ["Os Três Eixos"](https://ieeexplore.ieee.org/document/7163220#sec2e) que nos ajuda a entender por que não podemos atingir 100% de cada uma das seis metas simultaneamente.

Embora uma política de privacidade tradicionalmente proteja a maioria desses processos, essa abordagem nem sempre é ideal porque:

- Desenvolvedores não são especialistas jurídicos, mas ainda precisam estar em conformidade com a legislação.
- Os usuários quase sempre têm que ler políticas longas e prolixas.

### A Nova Abordagem (Google e Apple)

Para enfrentar esses desafios e informar melhor os usuários, Google e Apple introduziram novos sistemas de rotulagem de privacidade (muito na linha da proposta do NIST) para ajudar os usuários a entender facilmente como seus dados estão sendo coletados, tratados e compartilhados, [Rotulagem de Cibersegurança de Software de Consumo](https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.02042022-1.pdf). Suas abordagens podem ser vistas em:

- As [Etiquetas Nutricionais](https://www.apple.com/privacy/labels/) da App Store (desde 2020).
- A [Seção de Segurança de Dados](https://developer.android.com/guide/topics/data/collect-share) do Google Play (desde 2021).

Como este é um novo requisito em ambas as plataformas, esses rótulos devem ser precisos para tranquilizar os usuários e mitigar abusos.

### Programa Google ADA MASA

Como testes de segurança regulares ajudam os desenvolvedores a identificar vulnerabilidades-chave em seus aplicativos, o Google Play permitirá que desenvolvedores que concluíram validação de segurança independente informem os usuários divulgando esse fato na seção de Segurança de Dados do aplicativo. O compromisso do desenvolvedor com segurança e privacidade visa tranquilizar os usuários.

Como parte do processo para fornecer mais transparência na arquitetura de segurança do aplicativo, o Google introduziu o programa [MASA (Avaliação de Segurança de Aplicativos Móveis)](https://appdefensealliance.dev/masa) como parte da [App Defense Alliance (ADA)](https://appdefensealliance.dev/). Como o MASA é um padrão globalmente reconhecido para segurança de aplicativos móveis para o ecossistema de aplicativos móveis, o Google está reconhecendo a importância da segurança nesta indústria. Os desenvolvedores podem trabalhar diretamente com um parceiro de Laboratório Autorizado para iniciar uma avaliação de segurança que é validada independentemente contra um conjunto de requisitos MASVS Nível 1, e o Google reconhecerá esse esforço permitindo que eles divulguem esses testes na seção de Segurança de Dados do aplicativo.

<img src="Images/Chapters/0x04i/masa_framework.png" width="100%"/>

> Se você é um desenvolvedor e gostaria de participar, complete o [formulário de Revisão de Segurança Independente](https://docs.google.com/forms/d/e/1FAIpQLSdBl_eCNcUeUVDiB2duiJLZ5s4AV5AhDVuOz_1u8S9qhcXF5g/viewform "Google Play - Formulário de Revisão de Segurança Independente").

Claro que o teste é limitado e não garante segurança completa do aplicativo. A revisão independente pode não ter como escopo verificar a precisão e integridade das declarações de Segurança de Dados de um desenvolvedor, e os desenvolvedores permanecem exclusivamente responsáveis por fazer declarações completas e precisas na listagem de sua loja Play Store.

### Referências

Você pode aprender mais sobre este e outros tópicos relacionados à privacidade aqui:

- [Política de Privacidade de Aplicativo iOS](https://developer.apple.com/documentation/healthkit/protecting_user_privacy#3705073)
- [Seção de Detalhes de Privacidade iOS na App Store](https://developer.apple.com/app-store/app-privacy-details/)
- [Melhores Práticas de Privacidade iOS](https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy)
- [Política de Privacidade de Aplicativo Android](https://support.google.com/googleplay/android-developer/answer/9859455#privacy_policy)
- [Seção de Segurança de Dados Android no Google Play](https://support.google.com/googleplay/android-developer/answer/10787469)
- [Preparando seu aplicativo para a nova seção de segurança de dados no Google Play](https://www.youtube.com/watch?v=J7TM0Yy0aTQ)
- [Melhores Práticas de Privacidade Android](https://developer.android.com/privacy/best-practices)

## Testando Privacidade em Aplicativos Móveis

Testadores de segurança devem estar cientes da lista do Google Play de [violações comuns de privacidade](https://support.google.com/googleplay/android-developer/answer/10144311?hl=en-GB#1&2&3&4&5&6&7&87&9&zippy=%2Cexamples-of-common-violations) embora não seja exaustiva. Alguns dos exemplos estão abaixo:

- Exemplo 1: Um aplicativo que acessa o inventário de aplicativos instalados de um usuário e não trata esses dados como dados pessoais ou sensíveis, enviando-os pela rede (violando MSTG-STORAGE-4) ou para outro aplicativo via mecanismos IPC (violando MSTG-STORAGE-6).
- Exemplo 2: Um aplicativo exibe dados sensíveis, como detalhes de cartão de crédito ou senhas de usuário, sem autorização do usuário, por exemplo, biometria (violando MSTG-AUTH-10).
- Exemplo 3: Um aplicativo que acessa dados de telefone ou agenda de contatos de um usuário e não trata esses dados como dados pessoais ou sensíveis, além de enviá-los por uma conexão de rede não segura (violando MSTG-NETWORK-1).
- Exemplo 4: Um aplicativo coleta localização do dispositivo (que aparentemente não é necessária para seu funcionamento adequado) e não tem uma divulgação proeminente explicando qual recurso usa esses dados (violando MSTG-PLATFORM-1).

> Você pode encontrar mais [violações comuns na Ajuda do Google Play Console](https://support.google.com/googleplay/android-developer/answer/10144311?hl=en-GB#1&2&3&4&5&6&7&87&9&zippy=%2Cexamples-of-common-violations "Ajuda do Google Play Console - Exemplos de Violações Comuns de Privacidade") indo para **Centro de Políticas -> Privacidade, engano e abuso de dispositivo -> Dados do usuário**.

Como você pode esperar, essas categorias de teste estão relacionadas entre si. Quando você está testando elas, frequentemente está testando indiretamente a proteção da privacidade do usuário. Este fato permitirá que você forneça relatórios melhores e mais abrangentes. Muitas vezes você poderá reutilizar evidências de outros testes para testar a Proteção de Privacidade do Usuário).

## Testando Divulgação de Privacidade de Dados no Mercado de Aplicativos

Este documento está interessado apenas em determinar quais informações relacionadas à privacidade estão sendo divulgadas pelos desenvolvedores e discutir como avaliar essas informações para decidir se parecem razoáveis (da mesma forma que você faria ao testar permissões).

> Embora seja possível que os desenvolvedores não estejam declarando certas informações que de fato estão sendo coletadas e/ou compartilhadas, esse é um tópico para um teste diferente. Neste teste, você não deve fornecer garantia de violação de privacidade.

### Análise Estática

Para realizar uma análise estática, siga estas etapas:

1. Pesquise o aplicativo no mercado de aplicativos correspondente (por exemplo, Google Play, App Store).
2. Vá para a seção ["Detalhes de Privacidade"](https://developer.apple.com/app-store/app-privacy-details/) (App Store) ou ["Seção de Segurança"](https://developer.android.com/guide/topics/data/collect-share) (Google Play).
3. Determine se há alguma informação disponível.

O aplicativo passa no teste desde que o desenvolvedor tenha cumprido as diretrizes do mercado de aplicativos e incluído os rótulos e explicações necessários. As divulgações do desenvolvedor no mercado de aplicativos devem ser armazenadas como evidência, para que você possa usá-las posteriormente para determinar possíveis violações de privacidade ou proteção de dados.

### Análise Dinâmica

Como uma etapa opcional, você também pode fornecer algum tipo de evidência como parte deste teste. Por exemplo, se você está testando um aplicativo iOS, pode facilmente habilitar a gravação de atividade do aplicativo e exportar um [Relatório de Privacidade](https://developer.apple.com/documentation/network/privacy_management/inspecting_app_activity_data) que contém acesso detalhado do aplicativo a diferentes recursos, como fotos, contatos, câmera, microfone, conexões de rede, etc.

Uma análise dinâmica tem muitas vantagens para testar outras categorias MASVS e fornece informações muito úteis que você pode usar para [testar comunicação de rede](0x06g-Testing-Network-Communication.md) para MASVS-NETWORK ou ao [testar interação do aplicativo com a plataforma](0x06h-Testing-Platform-Interaction.md) para MASVS-PLATFORM. Ao testar essas outras categorias, você pode ter feito medições semelhantes usando outras ferramentas de teste. Você também pode fornecer isso como evidência para este teste.

> Embora as informações disponíveis devam ser comparadas com o que o aplicativo realmente deve fazer, isso estará longe de ser uma tarefa trivial que pode levar de vários dias a semanas para terminar, dependendo de seus recursos e das capacidades de suas ferramentas automatizadas. Esses testes também dependem muito da funcionalidade e contexto do aplicativo e devem ser idealmente realizados em uma configuração de caixa branca trabalhando muito de perto com os desenvolvedores do aplicativo.

## Testando Educação do Usuário sobre Melhores Práticas de Segurança

Determinar se o aplicativo educa os usuários e os ajuda a entender as necessidades de segurança é especialmente desafiador se você pretende automatizar o processo. Recomendamos usar o aplicativo extensivamente e tentar responder às seguintes perguntas sempre que aplicável:

- **Uso de impressão digital**: Quando impressões digitais são usadas para autenticação fornecendo acesso a transações/informações de alto risco,

    _o aplicativo informa o usuário sobre problemas potenciais ao ter múltiplas impressões digitais de outras pessoas registradas no dispositivo também?_

- **Rooting/jailbreaking**: Quando a detecção de root ou jailbreak é implementada,

    _o aplicativo informa o usuário do fato de que certas ações de alto risco carregarão risco adicional devido ao status de jailbroken/rooted do dispositivo?_

- **Credenciais específicas**: Quando um usuário obtém um código de recuperação, uma senha ou um pin do aplicativo (ou define um),

    _o aplicativo instrui o usuário a nunca compartilhar isso com mais ninguém e que apenas o aplicativo o solicitará?_

- **Distribuição de aplicativo**: No caso de um aplicativo de alto risco e para evitar que os usuários baixem versões comprometidas do aplicativo,

    _o fabricante do aplicativo comunica adequadamente a maneira oficial de distribuir o aplicativo (por exemplo, do Google Play ou da App Store)?_

- **Divulgação Proeminente**: Em qualquer caso,

    _o aplicativo exibe divulgação proeminente de acesso, coleta, uso e compartilhamento de dados? por exemplo, o aplicativo usa o [App Tracking Transparency Framework](https://developer.app