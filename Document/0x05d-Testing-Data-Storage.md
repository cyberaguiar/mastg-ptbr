---
masvs_category: MASVS-STORAGE
platform: android
---

# Armazenamento de Dados Android

## Visão Geral

Este capítulo discute a importância de proteger dados sensíveis, como tokens de autenticação e informações privadas, vitais para a segurança móvel. Veremos as APIs do Android para armazenamento local de dados e compartilharemos melhores práticas.

Embora seja preferível limitar dados sensíveis no armazenamento local, ou evitá-los sempre que possível, casos de uso práticos frequentemente necessitam do armazenamento de dados do usuário. Por exemplo, para melhorar a experiência do usuário, aplicativos armazenam em cache tokens de autenticação localmente, contornando a necessidade de entrada de senha complexa em cada inicialização do aplicativo. Aplicativos também podem precisar armazenar informações pessoalmente identificáveis (PII) e outros dados sensíveis.

Dados sensíveis podem se tornar vulneráveis se impropriamente protegidos, potencialmente armazenados em vários locais, incluindo o dispositivo ou um cartão SD externo. É importante identificar as informações processadas pelo aplicativo móvel e classificar o que conta como dados sensíveis. Confira a seção ["Identificando Dados Sensíveis"](0x04b-Mobile-App-Security-Testing.md#identifying-sensitive-data "Identificando Dados Sensíveis") no capítulo "Teste de Segurança de Aplicativos Móveis" para detalhes de classificação de dados. Consulte [Dicas de Segurança para Armazenar Dados](https://developer.android.com/training/articles/security-tips.html#StoringData "Dicas de Segurança para Armazenar Dados") no guia do desenvolvedor Android para insights abrangentes.

Riscos de divulgação de informações sensíveis incluem potencial descriptografia de informações, ataques de engenharia social (se PII for divulgada), sequestro de conta (se informações de sessão ou um token de autenticação forem divulgados) e exploração de aplicativo com uma opção de pagamento.

Além da proteção de dados, valide e sane dados de qualquer fonte de armazenamento. Isso inclui verificar tipos de dados corretos e implementar controles criptográficos, como HMACs, para integridade de dados.

O Android oferece vários métodos de [armazenamento de dados](https://developer.android.com/training/data-storage "Armazenando Dados no Android"), adaptados a usuários, desenvolvedores e aplicações. Técnicas comuns de armazenamento persistente incluem:

- Shared Preferences
- Bancos de Dados SQLite
- Bancos de Dados Firebase
- Bancos de Dados Realm
- Armazenamento Interno
- Armazenamento Externo
- Keystore

Além disso, outras funções do Android que podem resultar em armazenamento de dados e devem ser testadas incluem:

- Funções de Logging
- Backups Android
- Memória de Processos
- Caches de Teclado
- Screenshots

Entender cada função relevante de armazenamento de dados é crucial para realizar os casos de teste apropriados. Esta visão geral fornece um breve esboço desses métodos de armazenamento de dados e direciona testadores para documentação relevante adicional.