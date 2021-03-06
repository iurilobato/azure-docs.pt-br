Para criar uma Rede Virtual no modelo de implantação do Gerenciador de Recursos usando o portal do Azure, siga as etapas abaixo. As capturas de tela são fornecidas como exemplos. Substitua os valores pelos seus próprios. Para saber mais sobre como trabalhar com as redes virtuais, consulte a [Visão Geral da Rede Virtual](../articles/virtual-network/virtual-networks-overview.md)

1. Em um navegador, acesse o [Portal do Azure](http://portal.azure.com) e entre com sua conta do Azure.
2. Clique em **Novo**. No campo **Pesquisar no Marketplace**, digite "Rede Virtual". Localize **Rede Virtual** na lista retornada e clique para abrir a folha **Rede Virtual**.
3. Perto da parte inferior da folha Rede Virtual, na lista **Selecionar um modelo de implantação**, selecione **Gerenciador de Recursos** e clique em **Criar**.
4. Na folha **Criar rede virtual** , defina as configurações da VNet. Durante o preenchimento dos campos, o ponto de exclamação vermelho se tornará um ponto de seleção verde quando os caracteres digitados no campo forem válidos.
5. A folha **Criar rede virtual** é semelhante ao exemplo a seguir. Talvez alguns valores sejam preenchidos automaticamente. Se forem, substitua os valores pelos seus próprios.
   
    ![Folha Criar rede virtual](./media/vpn-gateway-basic-vnet-s2s-rm-portal-include/createvnet.png "Folha Criar rede virtual")
6. **Nome**: insira o nome de sua Rede Virtual.
7. **Espaço de endereço**: insira o espaço de endereço. Se houver vários espaços de endereço para adicionar, adicione seu espaço de endereço primeiro. Você pode adicionar outros espaços de endereço posteriormente, depois de criar a rede virtual. Verifique se o espaço de endereço especificado não se sobrepõe ao espaço de endereço de sua localização.
8. **Nome da sub-rede**: adicione o nome da sub-rede e o intervalo de endereços de sub-rede. Você pode adicionar outras sub-redes posteriormente, depois de criar a rede virtual.
9. **Assinatura**: verifique se a assinatura listada é a correta. Você pode alterar as assinaturas usando o menu suspenso.
10. **Grupo de recursos**: selecione um grupo de recursos existente ou crie um novo digitando um nome para seu novo grupo de recursos. Se você estiver criando um novo grupo, dê o nome do grupo de recursos de acordo com seus valores de configuração planejados. Para saber mais sobre grupos de recursos, visite [Visão geral do Gerenciador de Recursos do Azure](../articles/azure-resource-manager/resource-group-overview.md#resource-groups).
11. **Local**: selecione o local de sua rede virtual. O local determina onde ficarão os recursos que você implanta nessa rede virtual.
12. Selecione **Fixar no painel** se quiser encontrar sua VNet facilmente no painel. Em seguida, clique em **Criar**.
13. Depois de clicar em **Criar**, você verá um bloco em seu painel refletir o progresso de sua rede virtual. O bloco muda à medida que a rede virtual é criada.