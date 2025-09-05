---
masvs_category: MASVS-RESILIENCE
platform: ios
---

# Defesas Anti-Reversão iOS

## Visão Geral

Este capítulo cobre medidas de defesa em profundidade recomendadas para aplicativos que processam ou dão acesso a dados ou funcionalidades sensíveis. Pesquisas mostram que [muitos aplicativos da App Store frequentemente incluem essas medidas](https://seredynski.com/articles/a-security-review-of-1-300-appstore-applications "A security review of 1,300 AppStore applications - 5 April 2020").

Essas medidas devem ser aplicadas conforme necessário, com base em uma avaliação dos riscos causados por adulteração não autorizada do aplicativo e/ou engenharia reversa do código.

- Aplicativos nunca devem usar essas medidas como substituto para controles de segurança e, portanto, espera-se que atendam a outras medidas de segurança básicas, como o restante dos controles de segurança do MASVS.
- Aplicativos devem combinar essas medidas de forma inteligente em vez de usá-las individualmente. O objetivo é desencorajar engenheiros reversos de realizar análises adicionais.
- Integrar alguns dos controles em seu aplicativo pode aumentar a complexidade do seu aplicativo e até ter um impacto em seu desempenho.

Você pode aprender mais sobre princípios e riscos técnicos de engenharia reversa e modificação de código nestes documentos da OWASP:

- [OWASP Architectural Principles That Prevent Code Modification or Reverse Engineering](https://wiki.owasp.org/index.php/OWASP_Reverse_Engineering_and_Code_Modification_Prevention_Project "OWASP Architectural Principles That Prevent Code Modification or Reverse Engineering")
- [OWASP Technical Risks of Reverse Engineering and Unauthorized Code Modification](https://wiki.owasp.org/index.php/Technical_Risks_of_Reverse_Engineering_and_Unauthorized_Code_Modification "OWASP Technical Risks of Reverse Engineering and Unauthorized Code Modification")

**Aviso Geral:**

A **falta de qualquer uma dessas medidas não causa uma vulnerabilidade** - em vez disso, elas são destinadas a aumentar a resiliência do aplicativo contra engenharia reversa e ataques específicos do lado do cliente.

Nenhuma dessas medidas pode garantir 100% de eficácia, pois o engenheiro reverso sempre terá acesso total ao dispositivo e, portanto, sempre vencerá (dado tempo e recursos suficientes)!

Por exemplo, prevenir a depuração é virtualmente impossível. Se o aplicativo estiver publicamente disponível, ele pode ser executado em um dispositivo não confiável que está sob controle total do atacante. Um atacante muito determinado eventualmente conseguirá contornar todos os controles anti-depuração do aplicativo corrigindo o binário do aplicativo ou modificando dinamicamente o comportamento do aplicativo em tempo de execução com ferramentas como Frida.

As técnicas discutidas abaixo permitirão que você detecte várias maneiras pelas quais um atacante pode direcionar seu aplicativo. Como essas técnicas são documentadas publicamente, geralmente são fáceis de contornar. Usar técnicas de detecção de código aberto é um bom primeiro passo para melhorar a resiliência do seu aplicativo, mas ferramentas anti-detecção padrão podem contorná-las facilmente. Produtos comerciais normalmente oferecem maior resiliência, pois combinam várias técnicas, como:

- Usando técnicas de detecção não documentadas
- Implementando as mesmas técnicas de várias maneiras
- Acionando a lógica de detecção em diferentes cenários
- Fornecendo combinações de detecção únicas por build
- Trabalhando em conjunto com um componente de backend para verificação adicional e criptografia de payload HTTP
- Comunicando o status de detecção para o backend
- Ofuscamento estático avançado