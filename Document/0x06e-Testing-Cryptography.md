---
masvs_category: MASVS-CRYPTO
platform: ios
---

# APIs Criptográficas iOS

## Visão Geral

No capítulo ["Criptografia para Aplicativos Móveis"](0x04g-Testing-Cryptography.md), introduzimos as melhores práticas gerais de criptografia e descrevemos problemas típicos que podem ocorrer quando a criptografia é usada incorretamente. Neste capítulo, vamos entrar em mais detalhes sobre as APIs de criptografia do iOS. Mostraremos como identificar o uso dessas APIs no código fonte e como interpretar configurações criptográficas. Ao revisar código, certifique-se de comparar os parâmetros criptográficos usados com as melhores práticas atuais vinculadas neste guia.

A Apple fornece bibliotecas que incluem implementações da maioria dos algoritmos criptográficos comuns. O [Guia de Serviços Criptográficos da Apple](https://developer.apple.com/library/content/documentation/Security/Conceptual/cryptoservices/GeneralPurposeCrypto/GeneralPurposeCrypto.html "Apple Cryptographic Services Guide") é uma ótima referência. Ele contém documentação generalizada de como usar bibliotecas padrão para inicializar e usar primitivas criptográficas, informações úteis para análise de código fonte.