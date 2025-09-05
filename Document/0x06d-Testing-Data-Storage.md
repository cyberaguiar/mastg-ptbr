---
masvs_category: MASVS-STORAGE
platform: ios
---

# Armazenamento de Dados iOS

## Visão Geral

A proteção de dados sensíveis, como tokens de autenticação e informações privadas, é fundamental para a segurança móvel. Neste capítulo, você aprenderá sobre as APIs iOS para armazenamento local de dados e as melhores práticas para usá-las.

O mínimo possível de dados sensíveis deve ser salvo no armazenamento local permanente. No entanto, na maioria dos cenários práticos, pelo menos alguns dados do usuário devem ser armazenados. Felizmente, o iOS oferece APIs de armazenamento seguro, que permitem aos desenvolvedores usar o hardware criptográfico disponível em todos os dispositivos iOS. Se essas APIs forem usadas corretamente, dados e arquivos sensíveis podem ser protegidos via criptografia AES de 256 bits com suporte de hardware.