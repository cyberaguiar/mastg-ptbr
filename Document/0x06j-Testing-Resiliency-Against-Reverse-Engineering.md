---
masvs_category: MASVS-RESILIENCE
platform: ios
---

# Defesas Anti-Reversão no iOS

## Visão Geral

Este capítulo aborda medidas de defesa em profundidade recomendadas para apps que processam ou dão acesso a dados ou funcionalidades sensíveis. Pesquisas mostram que [muitos apps da App Store frequentemente incluem essas medidas](https://seredynski.com/articles/a-security-review-of-1-300-appstore-applications "A security review of 1,300 AppStore applications - 5 April 2020").

Essas medidas devem ser aplicadas conforme necessário, com base em uma avaliação dos riscos causados por adulteração não autorizada do app e/ou engenharia reversa do código.

- Os apps nunca devem usar essas medidas como substituição de controles de segurança, portanto espera-se que também cumpram outras medidas de segurança básicas do MASVS.
- Os apps devem combinar essas medidas de forma inteligente em vez de usá-las isoladamente. O objetivo é desencorajar engenheiros reversos de prosseguirem com a análise.
- Integrar alguns controles ao seu app pode aumentar a complexidade e até impactar seu desempenho.

Você pode aprender mais sobre princípios e riscos técnicos de engenharia reversa e modificação de código nestes documentos da OWASP:

- [OWASP Architectural Principles That Prevent Code Modification or Reverse Engineering](https://wiki.owasp.org/index.php/OWASP_Reverse_Engineering_and_Code_Modification_Prevention_Project "OWASP Architectural Principles That Prevent Code Modification or Reverse Engineering")
- [OWASP Technical Risks of Reverse Engineering and Unauthorized Code Modification](https://wiki.owasp.org/index.php/Technical_Risks_of_Reverse_Engineering_and_Unauthorized_Code_Modification "OWASP Technical Risks of Reverse Engineering and Unauthorized Code Modification")

**Aviso Geral:**

A **ausência de qualquer uma dessas medidas não causa uma vulnerabilidade** – elas servem para aumentar a resiliência do app contra engenharia reversa e ataques específicos no lado do cliente.

Nenhuma dessas medidas garante 100% de eficácia, já que o engenheiro reverso sempre terá acesso total ao dispositivo e, portanto, sempre vencerá (dado tempo e recursos suficientes)!

Por exemplo, impedir depuração é praticamente impossível. Se o app estiver disponível publicamente, ele pode ser executado em um dispositivo não confiável totalmente controlado pelo atacante. Um atacante muito determinado eventualmente conseguirá contornar todos os controles anti-debugging do app, seja modificando o binário ou alterando dinamicamente o comportamento em tempo de execução com ferramentas como Frida.

As técnicas discutidas abaixo permitem detectar várias maneiras pelas quais um atacante pode direcionar seu app. Como essas técnicas são documentadas publicamente, geralmente são fáceis de contornar. Usar técnicas de detecção open source é um bom primeiro passo para melhorar a resiliência do seu app, mas ferramentas de antideteção padrão podem contorná-las facilmente. Produtos comerciais normalmente oferecem resiliência maior, pois combinam múltiplas técnicas, como:

- Uso de técnicas de detecção não documentadas
- Implementação das mesmas técnicas de diferentes maneiras
- Acionamento da lógica de detecção em cenários diversos
- Combinações de detecção exclusivas por build
- Trabalho em conjunto com um componente backend para verificação adicional e criptografia de payload HTTP
- Comunicação do status de detecção para o backend
- Ofuscação estática avançada
