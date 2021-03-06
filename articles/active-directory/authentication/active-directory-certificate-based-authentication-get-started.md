---
title: Autenticação baseada em certificado-Azure Active Directory
description: Aprenda a configurar a autenticação baseada em certificado no seu ambiente
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: annaba
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: 6db7037cbcad335db77784ecfa624f08e88b1e83
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96744424"
---
# <a name="get-started-with-certificate-based-authentication-in-azure-active-directory"></a>Inicie com uma autenticação baseada em certificado do Azure Active Directory

A autenticação baseada em certificado permite que você seja autenticado pelo Azure Active Directory com um certificado de cliente em um dispositivo Windows, Android ou iOS ao conectar sua conta do Exchange Online com:

- Aplicativos móveis da Microsoft, como Microsoft Outlook e Microsoft Word
- Clientes do EAS (Exchange ActiveSync)

Configurar esse recurso elimina a necessidade de digitar uma combinação de nome de usuário e senha em determinados emails e aplicativos do Microsoft Office no seu dispositivo móvel.

Este tópico:

- Este tópico mostra como configurar e utilizar a autenticação baseada em certificado para usuários de locatários nos planos Office 365 corporativo, Business, educacional e governamental. Esse recurso está disponível na visualização em planos do Office 365 para China, defesa governamental e governo federal.
- Pressupõe que você já tem uma [infraestrutura de chave pública (PKI)](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831740(v=ws.11)) e [do AD FS](../hybrid/how-to-connect-fed-whatis.md) configurada.

## <a name="requirements"></a>Requisitos

Para configurar a autenticação baseada em certificado, as instruções a seguir devem ser verdadeiras:

- A CBA (autenticação baseada em certificado) só tem suporte em ambientes federados para aplicativos de navegador, clientes nativos que usam a ADAL (autenticação moderna) ou bibliotecas MSAL. A exceção é EAS (Exchange Active Sync) para EXO, que pode ser usada para contas gerenciadas e federadas.
- A autoridade de certificação raiz e qualquer autoridade de certificação intermediária devem ser configuradas no Azure Active Directory.
- Cada autoridade de certificação deve ter uma CRL (Lista de Certificados Revogados) que pode ser referenciada por meio de uma URL para a Internet.
- Você deve ter pelo menos uma autoridade de certificação configurada no Azure Active Directory. Você pode encontrar etapas relacionadas na seção [Configuração de autoridades de certificação](#step-2-configure-the-certificate-authorities).
- Para clientes do Exchange ActiveSync, o certificado do cliente deve ter o endereço de email roteável do usuário no Exchange Online no nome da entidade de segurança ou no valor do nome do RFC822 do campo nome alternativo da entidade. O Azure Active Directory mapeia o valor de RFC822 para o atributo Endereço de Proxy no diretório.
- O dispositivo do cliente deve ter acesso a pelo menos uma autoridade de certificação que emite certificados de cliente.
- Um certificado de cliente para autenticação de cliente deve ter sido emitido para seu cliente.

>[!IMPORTANT]
>O tamanho máximo de uma CRL para Azure Active Directory ser baixado com êxito e o cache é 20 MB, e o tempo necessário para baixar a CRL não deve exceder 10 segundos.  Se Azure Active Directory não puder baixar uma CRL, as autenticações baseadas em certificado usando certificados emitidos pela autoridade de certificação correspondente falharão. As práticas recomendadas para garantir que os arquivos de CRL estejam dentro das restrições de tamanho são manter os tempos de vida dos certificados dentro dos limites razoáveis e limpar os certificados expirados.

## <a name="step-1-select-your-device-platform"></a>Etapa 1: selecione a plataforma do dispositivo

Para a plataforma do dispositivo sob sua responsabilidade, a primeira etapa é examinar o seguinte:

- O suporte a aplicativos móveis do Office
- Os requisitos específicos de implementação

As informações relacionadas existentes para as seguintes plataformas de dispositivos:

- [Android](active-directory-certificate-based-authentication-android.md)
- [iOS](active-directory-certificate-based-authentication-ios.md)

## <a name="step-2-configure-the-certificate-authorities"></a>Etapa 2: Configuração de autoridades de certificação

Para configurar as autoridades de certificação no Azure Active Directory, carregue o seguinte para cada autoridade de certificação:

* A parte pública do certificado, no formato *.cer*
* As URLs para a Internet, em que as CRLs (Listas de Certificados Revogados) residem

Veja abaixo o esquema de uma autoridade de certificação:

```csharp
    class TrustedCAsForPasswordlessAuth
    {
       CertificateAuthorityInformation[] certificateAuthorities;
    }

    class CertificateAuthorityInformation

    {
        CertAuthorityType authorityType;
        X509Certificate trustedCertificate;
        string crlDistributionPoint;
        string deltaCrlDistributionPoint;
        string trustedIssuer;
        string trustedIssuerSKI;
    }

    enum CertAuthorityType
    {
        RootAuthority = 0,
        IntermediateAuthority = 1
    }
```

Para a configuração, você pode usar o [Azure Active Directory PowerShell versão 2](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0):

1. Inicie o Windows PowerShell com os privilégios de administrador.
2. Instale o módulo do Microsoft Azure AD versão [2.0.0.33](https://www.powershellgallery.com/packages/AzureAD/2.0.0.33) ou superior.

```powershell
    Install-Module -Name AzureAD –RequiredVersion 2.0.0.33
```

Como essa é a primeira etapa de configuração, você precisa estabelecer uma conexão com seu locatário. Como existe uma conexão com o seu locatário, você pode revisar, adicionar, excluir e modificar autoridades de certificação confiáveis que são definidas em seu diretório.

### <a name="connect"></a>Conectar

Para estabelecer uma conexão com seu locatário, use o cmdlet [Connect-AzureAD](/powershell/module/azuread/connect-azuread?view=azureadps-2.0):

```azurepowershell
    Connect-AzureAD
```

### <a name="retrieve"></a>Recuperar

Para recuperar as autoridades de certificação confiáveis que são definidas em seu diretório, use o cmdlet [Get-AzureADTrustedCertificateAuthority](/powershell/module/azuread/get-azureadtrustedcertificateauthority?view=azureadps-2.0).

```azurepowershell
    Get-AzureADTrustedCertificateAuthority
```

### <a name="add"></a>Adicionar

Para criar uma autoridade de certificação confiável, use o cmdlet [New-AzureADTrustedCertificateAuthority](/powershell/module/azuread/new-azureadtrustedcertificateauthority?view=azureadps-2.0) e defina o atributo **crlDistributionPoint** para um valor correto:

```azurepowershell
    $cert=Get-Content -Encoding byte "[LOCATION OF THE CER FILE]"
    $new_ca=New-Object -TypeName Microsoft.Open.AzureAD.Model.CertificateAuthorityInformation
    $new_ca.AuthorityType=0
    $new_ca.TrustedCertificate=$cert
    $new_ca.crlDistributionPoint="<CRL Distribution URL>"
    New-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $new_ca
```

### <a name="remove"></a>Remover

Para remover uma autoridade de certificação confiável, use o cmdlet [Remove-AzureADTrustedCertificateAuthority](/powershell/module/azuread/remove-azureadtrustedcertificateauthority?view=azureadps-2.0):

```azurepowershell
    $c=Get-AzureADTrustedCertificateAuthority
    Remove-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $c[2]
```

### <a name="modify"></a>Modificar

Para modificar uma autoridade de certificação confiável, use o cmdlet [Set-AzureADTrustedCertificateAuthority](/powershell/module/azuread/set-azureadtrustedcertificateauthority?view=azureadps-2.0):

```azurepowershell
    $c=Get-AzureADTrustedCertificateAuthority
    $c[0].AuthorityType=1
    Set-AzureADTrustedCertificateAuthority -CertificateAuthorityInformation $c[0]
```

## <a name="step-3-configure-revocation"></a>Etapa 3: configurar a revogação

Para revogar um certificado do cliente, o Azure Active Directory busca a CRL (Lista de Certificados Revogados) nas URLs carregadas como parte das informações da autoridade de certificado e a armazena em cache. O carimbo de data/hora da última publicação (propriedade **Effective Date** ) na CRL é usado para garantir que a CRL continua sendo válida. A CRL é referenciada periodicamente para revogar o acesso a certificados que fazem parte da lista.

Se uma revogação mais imediata for necessária (por exemplo, se um usuário perder um dispositivo), o token de autorização do usuário poderá ser invalidado. Para invalidar o token de autorização, defina o campo **StsRefreshTokenValidFrom** para esse usuário específico usando o Windows PowerShell. Você deve atualizar o campo **StsRefreshTokenValidFrom** para cada usuário do qual deseja revogar o acesso.

Para garantir que a revogação persista, é preciso definir **Effective Date** da CRL para uma data posterior ao valor definido por **StsRefreshTokenValidFrom** e garantir que o certificado em questão esteja na CRL.

As etapas a seguir descrevem o processo para atualizar e invalidar o token de autorização pela definição do campo **StsRefreshTokenValidFrom** .

**Para configurar a revogação:**

1. Conecte-se com credenciais de administrador ao serviço MSOL:

```powershell
        $msolcred = get-credential
        connect-msolservice -credential $msolcred
```

2. Recupere o valor StsRefreshTokensValidFrom atual para um usuário:

```powershell
        $user = Get-MsolUser -UserPrincipalName test@yourdomain.com`
        $user.StsRefreshTokensValidFrom
```

3. Configure um novo valor StsRefreshTokensValidFrom para o usuário igual ao carimbo de data/hora atual:

```powershell
        Set-MsolUser -UserPrincipalName test@yourdomain.com -StsRefreshTokensValidFrom ("03/05/2016")
```

A data que você define deve estar no futuro. Se a data não estiver no futuro, a propriedade **StsRefreshTokensValidFrom** não será definida. Se a data estiver no futuro, **StsRefreshTokensValidFrom** será definida para a hora atual (não a data indicada pelo comando Set-MsolUser).

## <a name="step-4-test-your-configuration"></a>Etapa 4: testar a sua configuração

### <a name="testing-your-certificate"></a>Teste do seu certificado

Como primeiro teste de configuração, entre no [Outlook Web Access](https://outlook.office365.com) ou no [SharePoint Online](https://microsoft.sharepoint.com) usando seu **navegador no dispositivo**.

Se a entrada for bem-sucedida, você saberá que:

- O certificado de usuário foi provisionado para o seu dispositivo de teste
- O AD FS está configurado corretamente

### <a name="testing-office-mobile-applications"></a>Testando aplicativos móveis do Office

**Para testar a autenticação baseada em certificado no seu aplicativo móvel do Office:**

1. No seu dispositivo de teste, instale um aplicativo móvel do Office (por exemplo, OneDrive).
3. Inicie o aplicativo.
4. Insira seu nome de usuário e escolha o certificado de usuário que deseja usar.

Você deve se conectar com êxito.

### <a name="testing-exchange-activesync-client-applications"></a>Testando aplicativos cliente do Exchange ActiveSync

Para acessar o Exchange ActiveSync (EAS) por meio da autenticação baseada em certificado, um perfil do EAS que contém o certificado de cliente deve estar disponível para o aplicativo.

O perfil do EAS deve conter as seguintes informações:

- O certificado do usuário a ser usado para autenticação

- O ponto de extremidade do EAS (por exemplo, outlook.office365.com)

Um perfil do EAS pode ser configurado e colocado no dispositivo por meio da utilização de um gerenciamento de dispositivo móvel (MDM), como o Intune, ou colocando o certificado manualmente no perfil do EAS no dispositivo.

### <a name="testing-eas-client-applications-on-android"></a>Testando os aplicativos cliente do EAS no Android

**Para testar a autenticação do certificado:**

1. Configure um perfil EAS no aplicativo que atenda aos requisitos da seção anterior.
2. Abra o aplicativo e verifique a sincronização de email.

## <a name="next-steps"></a>Próximas etapas

[Informações adicionais sobre autenticação baseada em certificado nos dispositivos Android.](active-directory-certificate-based-authentication-android.md)

[Informações adicionais sobre autenticação baseada em certificado nos dispositivos iOS.](active-directory-certificate-based-authentication-ios.md)