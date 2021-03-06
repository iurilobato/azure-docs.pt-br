### <a name="configure-a-network-security-group-inbound-rule-for-the-vm"></a>Configurar uma regra de entrada do Grupo de Segurança de Rede para a VM
Se você quiser ser capaz de se conectar ao SQL Server pela Internet, você precisa configurar uma regra de entrada no Grupo de Segurança de Rede para a porta que sua instância do SQL Server está escutando. Por padrão, essa é a porta TCP 1433.

1. No portal, selecione **Máquinas Virtuais**e selecione sua VM do SQL Server.
2. Selecione **Interfaces de rede**.
   
    ![interface de rede](./media/virtual-machines-sql-server-connection-steps/rm-network-interface.png)
3. Em seguida, selecione a Interface de rede para sua VM.
4. Clique no link **Grupo de segurança de rede** .
   
    ![interface de rede](./media/virtual-machines-sql-server-connection-steps/rm-network-security-group.png)
5. Nas propriedades do Grupo de Segurança de Rede, expanda **Regras de segurança de entrada**.
6. Clique no botão **Adicionar** .
7. Forneça um **Nome** de "SQLServerPublicTraffic".
8. Altere o **Protocol** para **TCP**.
9. Especifique um **Intervalo de Porta de Destino** de 1433 (ou a porta que sua instância do SQL Server estiver escutando).
10. Verifique se **Ação** está definida como **Permitir**. A caixa de diálogo da regra de segurança deve ter aparência semelhante à captura de tela a seguir.
    
     ![grupo de segurança de rede](./media/virtual-machines-sql-server-connection-steps/rm-network-security-rule.png)
11. Clique em **OK** para salvar a regra para a sua VM.

> [!NOTE]
> É possível ter um segundo Grupo de Segurança de Rede associado a sua sub-rede (isso é separado do grupo de segurança de rede na máquina virtual). Isso não é feito para você por padrão. No entanto, se você criou um grupo de segurança de rede na sub-rede, você deverá abrir a porta 1433 tanto da sub-rede quanto no Grupo de Segurança de Rede da VM. 
> 
> 

