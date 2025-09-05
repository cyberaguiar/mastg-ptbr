---
masvs_category: MASVS-RESILIENCE
platform: android
---

# Defesas Anti-Reversão Android

## Visão Geral

**Aviso Geral:**

A **falta de qualquer uma dessas medidas não causa uma vulnerabilidade** - em vez disso, elas são destinadas a aumentar a resiliência do aplicativo contra engenharia reversa e ataques específicos do lado do cliente.

Nenhuma dessas medidas pode garantir 100% de eficácia, pois o engenheiro reverso sempre terá acesso total ao dispositivo e, portanto, sempre vencerá (dado tempo e recursos suficientes)!

Por exemplo, prevenir a depuração é virtualmente impossível. Se o aplicativo estiver disponível publicamente, ele pode ser executado em um dispositivo não confiável que está sob controle total do atacante. Um atacante muito determinado eventualmente conseguirá contornar todos os controles anti-depuração do aplicativo aplicando patches no binário do aplicativo ou modificando dinamicamente o comportamento do aplicativo em tempo de execução com ferramentas como Frida.

Você pode aprender mais sobre os princípios e riscos técnicos da engenharia reversa e modificação de código nestes documentos da OWASP:

- [Princípios Arquiteturais da OWASP que Previnem Modificação de Código ou Engenharia Reversa](https://wiki.owasp.org/index.php/OWASP_Reverse_Engineering_and_Code_Modification_Prevention_Project "OWASP Architectural Principles That Prevent Code Modification or Reverse Engineering")
- [Riscos Técnicos da OWASP de Engenharia Reversa e Modificação de Código Não Autorizada](https://wiki.owasp.org/index.php/Technical_Risks_of_Reverse_Engineering_and_Unauthorized_Code_Modification "OWASP Technical Risks of Reverse Engineering and Unauthorized Code Modification")