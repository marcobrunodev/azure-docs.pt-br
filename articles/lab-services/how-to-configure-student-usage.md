---
title: Definir configurações de uso em laboratórios de Azure Lab Services
description: Saiba como configurar o número de alunos para um laboratório, obtê-los registrados com o laboratório, controlar o número de horas em que eles podem usar a VM e muito mais.
ms.topic: article
ms.date: 12/01/2020
ms.openlocfilehash: 3b05246445aea708312891ec631a35da3bc1eb8e
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602624"
---
# <a name="add-and-manage-lab-users"></a>Adicionar e gerenciar os usuários do laboratório

Este artigo descreve como adicionar usuários de aluno a um laboratório, registrá-los com o laboratório, controlar o número de horas adicionais que podem usar a VM (máquina virtual) e muito mais. 

Quando você adiciona usuários, por padrão, a opção **restringir acesso** é ativada e, a menos que estejam na lista de usuários, os alunos não podem se registrar no laboratório, mesmo que tenham um link de registro. Somente os usuários listados podem se registrar no laboratório usando o link de registro que você envia. Você pode desativar o **acesso restrito**, que permite que os alunos se registrem no laboratório, desde que tenham o link de registro. 

Este artigo mostra como adicionar usuários a um laboratório.

## <a name="add-users-from-an-azure-ad-group"></a>Adicionar usuários de um grupo do Azure AD

### <a name="overview"></a>Visão geral

Agora você pode sincronizar uma lista de usuários do laboratório a um grupo Azure Active Directory (Azure AD) existente para que não seja necessário adicionar ou excluir usuários manualmente. 

Um grupo do Azure AD pode ser criado dentro do Azure Active Directory da sua organização para gerenciar o acesso a recursos organizacionais e aplicativos baseados em nuvem. Para saber mais, confira [grupos do Azure ad](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-manage-groups). Se sua organização usa Microsoft Office 365 ou os serviços do Azure, sua organização já terá administradores que gerenciam seus Azure Active Directory. 

### <a name="sync-users-with-azure-ad-group"></a>Sincronizar usuários com o grupo do Azure AD

> [!IMPORTANT]
> Verifique se a lista de usuários está vazia. Se houver usuários existentes dentro de um laboratório que você adicionou manualmente ou por meio da importação de um arquivo CSV, a opção de sincronizar o laboratório com um grupo existente não será exibida. 

1. Entre no [site do Azure Lab Services](https://labs.azure.com/).
1. Selecione o laboratório com o qual você deseja trabalhar.
1. No painel esquerdo, selecione **Usuários**. 
1. Clique em **sincronizar do grupo**. 

    :::image type="content" source="./media/how-to-configure-student-usage/add-users-sync-group.png" alt-text="Adicionar usuários Sincronizando de um grupo do Azure AD":::
    
1. Você será solicitado a escolher um grupo existente do Azure AD para sincronizar seu laboratório. 
    
    Se você não vir um grupo do Azure AD na lista, isso pode ser devido aos seguintes motivos:

    -   Se você for um usuário convidado para um Azure Active Directory (geralmente, se você estiver fora da organização que possui o Azure AD), e não for possível procurar grupos dentro do Azure AD. Nesse caso, você não poderá adicionar um grupo do Azure AD ao laboratório nesse caso. 
    -   Os grupos do Azure AD criados por meio de equipes não aparecem nesta lista. Você pode adicionar o aplicativo Azure Lab Services dentro das equipes para criar e gerenciar laboratórios diretamente de dentro dele. Veja mais informações sobre como [gerenciar a lista de usuários de um laboratório de dentro das equipes](how-to-manage-user-lists-within-teams.md). 
1. Depois de escolher o grupo do Azure AD para sincronizar seu laboratório, clique em **Adicionar**.
1. Depois que um laboratório é sincronizado, ele efetua pull de todos dentro do grupo do Azure AD para o laboratório como usuários e você verá a lista de usuários atualizada. Somente as pessoas deste grupo do Azure AD terão acesso ao seu laboratório. A lista de usuários será atualizada a cada 24 horas para corresponder à associação mais recente do grupo do Azure AD. Você também pode clicar no botão Sincronizar na guia usuários para sincronizar manualmente com as alterações mais recentes no grupo do Azure AD.
1. Convide os usuários para seu laboratório clicando no botão **convidar tudo** , que enviará um email para todos os usuários com o link de registro para o laboratório. 

### <a name="automatic-management-of-virtual-machines-based-on-changes-to-the-azure-ad-group"></a>Gerenciamento automático de máquinas virtuais com base em alterações no grupo do Azure AD 

Depois que o laboratório for sincronizado com um grupo do Azure AD, o número de máquinas virtuais no laboratório corresponderá automaticamente ao número de usuários no grupo. Não será mais possível atualizar manualmente a capacidade do laboratório. Quando um usuário é adicionado ao grupo do Azure AD, um laboratório adicionará automaticamente uma máquina virtual para esse usuário. Quando um usuário é excluído do grupo do Azure AD, um laboratório excluirá automaticamente a máquina virtual do usuário do laboratório. 

## <a name="add-users-manually-from-emails-or-csv-file"></a>Adicionar usuários manualmente de email (s) ou arquivo CSV

Nesta seção, você adicionará os alunos manualmente (por endereço de email ou carregando um arquivo CSV). 

### <a name="add-users-by-email-address"></a>Adicionar usuários por endereço de email

1. No painel esquerdo, selecione **Usuários**. 
1. Clique em **Adicionar usuários manualmente**. 

    :::image type="content" source="./media/how-to-configure-student-usage/add-users-manually.png" alt-text="Adicionar usuários manualmente":::
1. Selecione **Adicionar por endereço de email** (padrão), insira os endereços de email dos alunos em linhas separadas ou em uma única linha separada por ponto e vírgula. 

    :::image type="content" source="./media/how-to-configure-student-usage/add-users-email-addresses.png" alt-text="Adicionar endereços de email dos usuários":::
1. Selecione **Salvar**. 

    A lista exibe os endereços de email e os status dos usuários atuais, se eles estão registrados no laboratório ou não. 

    :::image type="content" source="./media/how-to-configure-student-usage/list-of-added-users.png" alt-text="Lista de usuários":::

    > [!NOTE]
    > Depois que os alunos são registrados com o laboratório, a lista exibe seus nomes. O nome que é mostrado na lista é construído usando o nome e o sobrenome dos alunos em Azure Active Directory. 

### <a name="add-users-by-uploading-a-csv-file"></a>Adicionar usuários ao carregar um arquivo CSV

Você também pode adicionar usuários carregando um arquivo CSV que contém seus endereços de email. 

Um arquivo de texto CSV é usado para armazenar dados tabulares separados por vírgula (CSV) (números e texto). Em vez de armazenar informações em campos de colunas (como em planilhas), um arquivo CSV armazena informações separadas por vírgulas. Cada linha em um arquivo CSV terá o mesmo número de campos separados por vírgulas ". Você pode usar o Excel para criar e editar arquivos CSV com facilidade.

1. No Microsoft Excel, crie um arquivo CSV que liste os endereços de email dos alunos em uma coluna.

    :::image type="content" source="./media/how-to-configure-student-usage/csv-file-with-users.png" alt-text="Lista de usuários em um arquivo CSV":::
1. Na parte superior do painel **usuários** , selecione **Adicionar usuários** e, em seguida, selecione **carregar CSV**.
1. Selecione o arquivo CSV que contém os endereços de email dos alunos e, em seguida, selecione **abrir**.

    A janela **Adicionar usuários** exibe a lista de endereços de email do arquivo CSV. 
1. Selecione **Salvar**. 
1. No painel **usuários** , exiba a lista de alunos adicionados. 

    :::image type="content" source="./media/how-to-configure-student-usage/list-of-added-users.png" alt-text="Lista de usuários adicionados no painel de usuários":::

## <a name="send-invitations-to-users"></a>Enviar convites aos usuários

Para enviar um link de registro para novos usuários, use um dos métodos a seguir. 

Se a opção de **acesso restrito** estiver habilitada para o laboratório, somente os usuários listados poderão usar o link de registro para se registrar no laboratório. Essa opção é habilitada por padrão. 

### <a name="invite-all-users"></a>Convidar todos os usuários

Esse método mostra como enviar emails com um link de registro e uma mensagem opcional para todos os alunos listados.

1. No painel **usuários** , selecione **convidar todos**. 

    ![O botão "convidar tudo"](./media/tutorial-setup-classroom-lab/invite-all-button.png)

1. Na janela **Enviar convite por email** , insira uma mensagem opcional e, em seguida, selecione **Enviar**. 

    O email inclui automaticamente o link de registro. Para obter e salvar o link de registro separadamente, selecione as reticências (**...**) na parte superior do painel **usuários** e, em seguida, selecione **link de registro**. 

    ![A janela "enviar link de registro por email"](./media/tutorial-setup-classroom-lab/send-email.png)

    A coluna **convite** da lista **usuários** exibe o status do convite para cada usuário adicionado. O status deve ser alterado para **enviando** e, em seguida, para **\<date> enviado**. 

### <a name="invite-selected-users"></a>Convidar usuários selecionados

Esse método mostra como convidar apenas determinados alunos e obter um link de registro que você pode compartilhar com outras pessoas.

1. No painel **usuários** , selecione um aluno ou vários alunos na lista. 

1. Na linha do aluno que você selecionou, selecione o ícone de **envelope** ou, na barra de ferramentas, selecione **convidar**. 

    ![Convidar usuários selecionados](./media/how-to-configure-student-usage/invite-selected-users.png)

1. Na janela **Enviar convite por email** , insira uma **mensagem** opcional e, em seguida, selecione **Enviar**. 

    ![Enviar email para os usuários selecionados](./media/how-to-configure-student-usage/send-invitation-to-selected-users.png)

    O painel **usuários** exibe o status dessa operação na coluna **convite** da tabela. O email de convite inclui o link de registro que os alunos podem usar para se registrarem no laboratório.

## <a name="get-the-registration-link"></a>Obter o link de registro

Nesta seção, você pode obter o link de registro no portal e enviá-lo usando seu próprio aplicativo de email. 

1. No painel **usuários** , selecione **link de registro**.

    ![Link de registro do aluno](./media/how-to-configure-student-usage/registration-link-button.png)

1. Na janela **registro do usuário** , selecione **copiar** e, em seguida, selecione **concluído**. 

    ![A janela "registro do usuário"](./media/how-to-configure-student-usage/registration-link.png)

    O link é copiado para a área de transferência. 
    
1. Em seu aplicativo de email, Cole o link de registro e envie o email para um aluno para que o aluno possa se registrar para a classe. 

## <a name="view-registered-users"></a>Exibir usuários registrados

1. Vá para o site [Azure Lab Services](https://labs.azure.com) . 
1. Selecione **entrar** e insira suas credenciais. O Azure Lab Services oferece suporte a contas organizacionais e contas Microsoft.
1. Na página **meus laboratórios** , selecione o laboratório cujo uso você deseja controlar. 
1. No painel esquerdo, selecione **usuários** ou selecione o bloco **usuários** . 

    O painel **usuários** exibe uma lista de alunos que se registraram com seu laboratório.  

    ![Lista de usuários registrados](./media/tutorial-track-usage/registered-users.png)

## <a name="set-quotas-for-users"></a>Definir cotas para usuários

Você pode definir uma cota de hora para cada aluno fazendo o seguinte: 

1. No painel **usuários** , selecione **cota por usuário: \<number> hora (s)** na barra de ferramentas. 
1. Na janela **cota por usuário** , especifique o número de horas que você deseja dar a cada aluno fora da hora da classe agendada e, em seguida, selecione **salvar**.

    ![A janela "cota por usuário"](./media/how-to-configure-student-usage/quota-per-user.png)    

    Os valores alterados agora são exibidos no botão **cota por usuário: \<number of hours>** na barra de ferramentas e na lista de usuários, conforme mostrado aqui:

    ![Horas de cota por usuário](./media/how-to-configure-student-usage/quot-per-user-after.png)

    > [!IMPORTANT]
    > O [tempo de execução agendado das VMs](how-to-create-schedules.md) não conta a cota que é alocada para um aluno. A cota é para o tempo fora das horas agendadas que um aluno gasta nas VMs. 

## <a name="set-additional-quotas-for-specific-users"></a>Definir cotas adicionais para usuários específicos

Você pode especificar cotas para determinados alunos além das cotas comuns que foram definidas para todos os usuários na seção anterior. Por exemplo, se você, como instrutor, definir a cota para todos os alunos como 10 horas e definir uma cota adicional de 5 horas para um aluno específico, esse aluno receberá 15 (10 + 5) horas de cota. Se você alterar a cota comum posteriormente para, digamos, 15, o aluno receberá 20 (15 + 5) horas de cota. Lembre-se de que essa cota geral está fora do horário agendado. O tempo que um aluno gasta em uma VM de laboratório durante o horário agendado não conta contra essa cota. 

Para definir cotas adicionais, faça o seguinte:

1. No painel **usuários** , selecione um aluno na lista e, em seguida, selecione **ajustar cota** na barra de ferramentas. 

    ![O botão "ajustar cota"](./media/how-to-configure-student-usage/adjust-quota-button.png)

1. Em **ajustar cota para \<selected user or users email address>**, insira o número de horas de laboratório adicionais que você deseja conceder ao aluno ou aos alunos selecionados e, em seguida, selecione **aplicar**. 

    ![A "ajustar cota..." Window](./media/how-to-configure-student-usage/additional-quota.png)

    A coluna **uso** exibe a cota atualizada para os alunos selecionados. 

    ![Novo uso para o usuário](./media/how-to-configure-student-usage/new-usage-hours.png)

## <a name="student-accounts"></a>Contas de alunos

Para adicionar alunos a um laboratório de sala de aula, você usa suas contas de email. Os alunos podem ter os seguintes tipos de contas de email:

- Uma conta de email de aluno fornecida pela instância de Azure Active Directory da sua universidade.
- Uma conta de email do domínio da Microsoft, como *Outlook.com*, *hotmail.com*, *MSN.com* ou *Live.com*.
- Uma conta de email que não seja da Microsoft, como uma fornecida pelo Yahoo! ou Google. No entanto, esses tipos de contas devem ser vinculados a um conta Microsoft.
- Uma conta do GitHub. Essa conta deve ser vinculada a um conta Microsoft.

### <a name="use-a-non-microsoft-email-account"></a>Usar uma conta de email que não seja da Microsoft

Os alunos podem usar contas de email que não são da Microsoft para se registrar e entrar em um laboratório de sala de aula.  No entanto, o registro requer que eles primeiro criem um conta Microsoft que esteja vinculado ao seu endereço de email que não seja da Microsoft.

Muitos alunos podem já ter uma conta Microsoft vinculada ao seu endereço de email que não seja da Microsoft. Por exemplo, os alunos já têm um conta Microsoft se usaram seu endereço de email com outros produtos ou serviços da Microsoft, como Office, Skype, OneDrive ou Windows.  

Quando os alunos usam o link de registro para entrar em uma sala de aula, eles são solicitados a fornecer seu endereço de email e senha. Os alunos que tentarem entrar com um não conta Microsoft que não esteja vinculado a uma conta Microsoft receberão a seguinte mensagem de erro: 

![Mensagem de erro ao entrar](./media/how-to-configure-student-usage/cant-find-account.png)

Aqui está um link para que os alunos [se inscrevam em um conta Microsoft](http://signup.live.com).  

> [!IMPORTANT]
> Quando os alunos entram em um laboratório de sala de aula, eles não recebem a opção de criar um conta Microsoft. Por esse motivo, recomendamos que você inclua esse link de inscrição, http://signup.live.com no email de registro de laboratório da sala de aula enviado aos alunos que estão usando contas que não são da Microsoft.

### <a name="use-a-github-account"></a>Usar uma conta do GitHub

Os alunos também podem usar uma conta do GitHub existente para registrar e entrar em um laboratório de sala de aula. Se eles já tiverem um conta Microsoft vinculado à sua conta do GitHub, os alunos poderão entrar e fornecer sua senha, conforme mostrado na seção anterior. 

Se eles ainda não tiverem vinculado sua conta do GitHub a um conta Microsoft, eles poderão fazer o seguinte:

1. Selecione o link **Opções de entrada** , como mostrado aqui:

    ![O link "opções de entrada"](./media/how-to-configure-student-usage/signin-options.png)

1. Na janela **Opções de entrada** , selecione **entrar com o GitHub**.

    ![O link "entrar com o GitHub"](./media/how-to-configure-student-usage/signin-github.png)

    No prompt, os alunos criam um conta Microsoft que está vinculado à sua conta do GitHub. A vinculação ocorre automaticamente quando selecionamos **Avançar**. Em seguida, eles são imediatamente conectados ao laboratório da sala de aula.

## <a name="export-a-list-of-users-to-a-csv-file"></a>Exportar uma lista de usuários para um arquivo CSV

1. Vá para o painel **usuários** .
1. Na barra de ferramentas, selecione as reticências (**...**) e, em seguida, selecione **Exportar CSV**. 

    ![O botão "exportar CSV"](./media/how-to-export-users-virtual-machines-csv/users-export-csv.png)

## <a name="next-steps"></a>Próximas etapas

Veja os artigos a seguir:

- Para administradores: [criar e gerenciar contas de laboratório](how-to-manage-lab-accounts.md)
- Para proprietários de laboratório: [criar e gerenciar laboratórios](how-to-manage-classroom-labs.md) e [configurar e publicar modelos](how-to-create-manage-template.md)
- Para usuários de laboratório: [laboratórios de acesso](how-to-use-classroom-lab.md)
