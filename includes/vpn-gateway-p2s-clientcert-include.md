Cada computador cliente que se conecta a uma rede virtual usando ponto a site deve ter um certificado de cliente instalado. O certificado de cliente é gerado a partir do certificado raiz e instalado em cada computador cliente. Se um certificado de cliente válido não for instalado, e o cliente tentar se conectar à rede virtual, a autenticação falhará.

Você pode gerar um certificado exclusivo para cada cliente ou pode usar o mesmo certificado para vários clientes. A vantagem da geração de certificados de cliente exclusivos é a capacidade de revogar um único certificado. Caso contrário, se vários clientes estiverem usando o mesmo certificado de cliente e for necessário revogá-lo, você precisará gerar e instalar novos certificados para todos os clientes que usam aquele certificado para autenticação.

Você pode gerar certificados de cliente usando os seguintes métodos:

- **Certificado corporativo:**

  - Se você estiver usando uma solução de certificado corporativo, gere um certificado de cliente com o formato de valor de nome comum 'name@yourdomain.com', em vez do formato 'nome do domínio\nomedeusuário'.
  - Verifique se o certificado do cliente baseia-se no modelo de certificado 'Usuário' que tem 'Autenticação de cliente' como o primeiro item na lista de uso, em vez de Logon de Cartão Inteligente, etc. Você pode verificar o certificado clicando duas vezes no certificado do cliente e exibindo *Detalhes > Uso Avançado de Chave*.

- **Certificado autoassinado:** Se você gerar um certificado de cliente de um certificado autoassinado usando as instruções do artigo [Criar um certificado raiz autoassinado para conexões Ponto a Site](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md#clientcert), ele será instalado automaticamente no computador que você usou para gerá-lo. Se você quiser instalar um certificado de cliente em outro computador cliente, será necessário exportá-lo. Siga as instruções no artigo para [exportar o certificado](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md#clientexport).