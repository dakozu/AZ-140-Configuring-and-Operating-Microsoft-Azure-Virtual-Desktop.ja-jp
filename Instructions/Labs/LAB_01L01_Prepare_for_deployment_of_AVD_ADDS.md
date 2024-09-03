---
lab:
  title: 'ラボ: Azure Virtual Desktop のデプロイを準備する (AD DS)'
  module: 'Module 1: Plan an AVD Architecture'
---

# ラボ - Azure Virtual Desktop のデプロイを準備する (AD DS)
# 受講生用ラボ マニュアル

## ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプションの所有者または共同作成者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウントと、Azure サブスクリプションに関連付けられた Microsoft Entra テナントのグローバル管理者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウント。

## 推定所要時間

60 分

## ラボのシナリオ

Active Directory Domain Services (AD DS) 環境の展開を準備する必要があります

## 目標
  
このラボを完了すると、次のことができるようになります。

- Azure VM を使用して Active Directory Domain Services (AD DS) 単一ドメイン フォレストをデプロイする
- AD DS フォレストを Microsoft Entra テナントに統合する

## ラボ ファイル

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json

## 手順

### 演習 0: vCPU クォータの数を増やす

この演習の主なタスクは次のとおりです。

1. 現在の vCPU 使用率を特定する
1. vCPU クォータの増加を要求する

#### タスク 1: 現在の vCPU 使用率を特定する

1. ラボ コンピューターから Web ブラウザーを起動して [Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. Azure portal で、検索テキスト ボックスのすぐ右にあるツール バー アイコンを選択して [**Cloud Shell**] ウィンドウを開きます。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、 **[PowerShell]** を選択します。 

   >**注**: **Cloud Shell** を初めて起動し、[**ストレージがマウントされていません**] というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。 

1. **Microsoft.Compute** および **Microsoft.Network** リソース プロバイダーが登録されていない場合は、Azure portal の **Cloud Shell** の PowerShell セッションで次を実行して、それらを登録します。

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Network'
   ```

1. Azure portal の **Cloud Shell** の PowerShell セッションで、以下を実行して、**Microsoft.Compute** リソース プロバイダーの登録ステータスを確認します。

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**注**: RegistrationState が "**Registered**" と表示されていることを確認します。 そうでない場合は、数分待ってからこの手順を繰り返します。

1. Azure portal の **[Cloud Shell]** の [PowerShell] セッションで、次のコマンドの場所を設定するために以下を実行します（たとえば `eastus` のように、`<Azure_region>` のプレースホルダーをこのラボで使用する Azure リージョンの名前に置き換えます）。

   ```powershell
   $location = '<Azure_region>'
   ```

1. Azure portal 内の **[Cloud Shell]** の [PowerShell] セッションで、次を実行して、**StandardDSv3Family** と **StandardBSFamily** のAzure VM での vCPU の現在の使用量と対応する制限を確認します。 

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

   > **注**: Azure リージョンの名前を識別するには、**Cloud Shell** の PowerShell プロンプトで `(Get-AzLocation).Location` を実行します。
   
1. 前の手順で実行したコマンドの出力を確認し、ターゲット Azure リージョン内の Azure VM の **Standard DSv3 Family vCPUs** に、少なくとも **30** 個の使用可能な vCPU があることを確認します。 既にそうである場合は、次の演習に直接進みます。 それ以外の場合は、この演習の次のタスクに進みます。 

#### タスク 2: vCPU クォータの引き上げを要求する

1. Azure portal で、「**サブスクリプション**」を検索して選択し、**[サブスクリプション]** ウィンドウから、このラボで使用する予定の Azure サブスクリプションを表すエントリを選択します。
1. Azure portal のサブスクリプション ブレードの左側の垂直メニューの **[設定]** セクションで、**[使用量 + クォータ]** を選択します。 

   **注**: クォータを増やすためにサポート チケットを作成する必要はない場合があります。

   **注:** クォータの引き上げを要求するには、多要素認証 (MFA) を使用したサインインが必要です。 MFA を使用してアカウントを構成する必要がある場合は、「[Azure Active Directory の多要素認証のデプロイを計画する](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-mfa-getstarted)」を参照してください。 
   
1. **[Azure Pass - スポンサー プラン | 使用量 + クォータ]** ブレードで **[リージョン]** を選択し、ドロップダウン リストで、このラボで使用する Azure リージョンの名前の横にあるチェック ボックスをオンにし、**[適用]** を選択して、**[コンピューティング]** エントリが **[リージョン]** エントリの左側にあるドロップダウン リストに表示されていることを確認し、検索ボックスに **「Standard DSv3」** と入力します。 
1. 結果の一覧で、**[Standard DSv3 Family vCPUs]** 項目の横にあるチェック ボックスをオンにし、ツール バーの **[クォータの引き上げの要求]** エントリを選択し、ドロップダウン リストで **[新しい制限を入力してください]** を選択します。
1. **[クォータの増加を要求]** ペインの **[新しい制限]** 列のテキスト ボックスに「**30**」と入力し、**[送信]** を選択します。
1. メッセージが表示されたら、**[クォータの増加を要求する]** ペインで **[多要素認証による認証]** を選択し、メッセージに従って認証します。
1. クォータの要求が完了するまで待ちます。  しばらくすると、**[クォータの詳細]** ブレードに、要求が承認され、クォータが増加したことが示されます。 **[クォータの詳細]** ブレードを閉じます。

   >**注**: Azure リージョンの選択と現在の需要によっては、サポート要求の提出が必要になる場合があります。 サポート要求の作成プロセスに関する手順については、「[Azure サポート要求を作成する](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request)」を参照してください。

### 演習 1: Active Directory Domain Services (AD DS) ドメインをデプロイする

この演習の主なタスクは次のとおりです。

1. Azure VM のデプロイを準備します
1. Azure Resource Manager クイックスタート テンプレートを使用してAD DS ドメイン コントローラーを実行する Azure VM をデプロイする
1. Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする
1. Azure Bastion をデプロイする

#### タスク 1: Azure VM のデプロイを準備する

1. ラボのコンピューターから Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動して、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. Azure portal で、Azure ポータル ページの上部にある **[リソース、サービス、ドキュメントを検索する]** テキスト ボックスを使用し、**[Microsoft Entra ID]** ブレードを検索してそこに移動します。
1. Microsoft Entra テナントの **[概要]** ブレードの左側にある縦型メニューの **[管理]** セクションで、**[プロパティ]** をクリックします。
1. Microsoft Entra テナントの **[プロパティ]** ブレードの一番下で、**[セキュリティの既定値群の管理]** リンクを選択します。
1. **[セキュリティの既定値の有効化]** ブレードで、必要に応じて、**[無効 (推奨されません)]** を選択し、**[組織で条件付きアクセスの使用を計画している]** オプション ボタンを選択して、**[保存]** を選択したら、**[無効]** を選択します。
1. Azure portal で、検索テキスト ボックスのすぐ右にあるツール バー アイコンを選択して **[Cloud Shell]** ペインを開きます。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、 **[PowerShell]** を選択します。 

   >**注**: **Cloud Shell** を初めて起動し、[**ストレージがマウントされていません**] というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。 


#### タスク 2: Azure Resource Manager クイックスタート テンプレートを使用して AD DS ドメイン コントローラーを実行する Azure VM をデプロイする

1. ラボ コンピューターの Azure portal を表示している Web ブラウザーで、Cloud Shell ペインの PowerShell セッションから、以下を実行してリソース グループを作成します (`<Azure_region>` プレースホルダーは、このラボで使うつもりの Azure リージョンの名前に置き換えます。例: `eastus`)。

   ```powershell
   $location = '<Azure_region>'
   $resourceGroupName = 'az140-11-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. Azure portal で、**[Cloud Shell]**] ペインを閉じます。
1. ラボ コンピューターから、同じ Web ブラウザー ウィンドウで、別の Web ブラウザー タブを開き、「[新しい Windows VM を作成し、新しい AD フォレスト、ドメイン、DC を作成する](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain)」という名前のクイック スタート テンプレートのカスタマイズされたバージョンに移動します。 
1. **[新しい Windows VM を作成し、新しい AD フォレスト、ドメイン、DC を作成する]** ページで、ページを下方向にスクロールして、**[Azure にデプロイ]** を選択します。 これにより、ブラウザーが Azure portal の **[新しい AD フォレストで Azure VM を作成する]** ブレードに自動的にリダイレクトされます。
1. **[新しい AD フォレストを使用して Azure VM を作成する]** ブレードで、**[パラメーターの編集]** を選択します。
1. **[パラメーターの編集]** ブレードで、**[ファイルの読み込み]** を選択し、**[開く]** ダイアログ ボックスで、**\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json** を選択して、**[開く]** を選択してから、**[保存]** を選択します。 
1. **[新しい AD フォレストで Azure VM を作成する]** ブレードで、次の設定を指定します (他の設定は既存の値のままにします)。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**az140-11-RG**|
   |ドメイン名|**adatum.com**|

1. **[新しい AD フォレストを使用して Azure VM を作成する]** ブレードで、**[確認と作成]** を選択し、**[作成]** を選択します。

   > **注**: このデプロイが完了するまで待ってから、次の演習に進んでください。 デプロイには 20 から 25 分かかる場合があります。 

#### タスク 1: Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする

1. ラボ コンピューターの Azure portal を表示する Web ブラウザーで、Cloud Shell ペインで PowerShell セッションを開き、以下を実行して、前のタスクで作成した **az140-adds-vnet11** という名前の仮想ネットワークに **cl-Subnet** という名前のサブネットを追加します。

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.0.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Azure portal の Cloud Shell ペインのツール バーで、**[ファイルのアップロード/ダウンロード]** アイコンを選択し、ドロップダウン メニューで **[アップロード]** を選択して、ファイル **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json** と **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json** を Cloud Shell のホーム ディレクトリにアップロードします。
1. [Cloud Shell] ペインの [PowerShell] セッションから、次を実行して、新しく作成されたサブネットにクライアントとして機能する Windows 10 を実行する Azure VM をデプロイします。

   ```powershell
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11.parameters.json
   ```

   > **注**: デプロイが完了するのを待たずに、次のタスクに進んでください。 デプロイには約 10 分かかります。

#### タスク 2: Azure Bastion をデプロイする 

> **注**: Azure Bastion を使用すると、この演習の前のタスクでデプロイしたパブリック エンドポイントを使用せずに Azure VM に接続できると同時に、オペレーティング システム レベルの資格情報を対象とするブルート フォース攻撃から保護することができます。

> **注**: ブラウザーでポップアップ機能が有効になっていることを確認します。

1. Azure portal が表示されているブラウザー ウィンドウで、別のタブを開き、そのブラウザー タブで [Azure portal](https://portal.azure.com) に移動します。
1. Azure portal で、検索テキスト ボックスのすぐ右にあるツール バー アイコンを選択して **[Cloud Shell]** ペインを開きます。
1. Cloud Shell ペインの PowerShell セッションから以下を実行して、この演習で先ほど作成した **az140-adds-vnet11** という名前の仮想ネットワークに **AzureBastionSubnet** という名前のサブネットを追加します。

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.0.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. [Cloud Shell] ペインを閉じます。
1. Azure portal で **[複数の要塞]** を検索して選択し、**[複数の要塞]** ブレードから **[+ 追加]** を選択します。
1. **[Bastion の作成]** ウィンドウの **[基本]** タブで、次の設定を指定して、**[確認および作成]** を選びます。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**az140-11-RG**|
   |名前|**az140-11-bastion**|
   |リージョン|この演習の前のタスクでリソースをデプロイしたのと同じ Azure リージョン|
   |レベル|**Basic**|
   |仮想ネットワーク|**az140-adds-vnet11**|
   |Subnet|**AzureBastionSubnet (10.0.254.0/24)**|
   |パブリック IP アドレス|**新規作成**|
   |パブリック IP の名前|**az140-adds-vnet11-ip**|

1. **[Bastion の作成]** ウィンドウの **[確認と作成]** タブで、**[作成]** を選択します。

   > **注**: このデプロイが完了するまで待ってから、次の演習に進んでください。 デプロイには約 10 分かかります。

### 演習 2: 単一の AD DS フォレストを単一の Microsoft Entra テナントに統合する
  
この演習の主なタスクは次のとおりです。

1. Microsoft Entra に同期される AD DS ユーザーとグループを作成する
1. AD DS UPN サフィックスを構成する
1. Microsoft Entra との同期を構成するために使用する Microsoft Entra ユーザーを作成する
1. Microsoft Entra Connect をインストールする

#### タスク 1: Microsoft Entra に同期される AD DS ユーザーとグループを作成する

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザーで、「**仮想マシン**」を検索して選択し、**[仮想マシン]** ブレードから **az140-dc-vm11** を選択します。
1. **[az140-dc-vm11]** ブレードで **[接続]** を選択し、ドロップダウン メニューで **[Bastion 経由で接続する]** を選択します。
1. プロンプトが表示されたら、次の資格情報を入力し、**[接続]** を選択します。

   |設定|値|
   |---|---|
   |[ユーザー名]|**Student**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-dc-vm11** への Bastion セッション内で、管理者として **Windows PowerShell ISE** を起動します。
1. **[管理者: Windows PowerShell ISE]** スクリプト ペインから、以下を実行して、管理者向け Internet Explorer のセキュリティ強化を無効にします。

   ```powershell
   $adminRegEntry = 'HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}'
   Set-ItemProperty -Path $AdminRegEntry -Name 'IsInstalled' -Value 0
   Stop-Process -Name Explorer
   ```

1. **[Administrator: Windows PowerShell ISE]** コンソールから、次を実行して、このラボで使用する Microsoft Entra テナントへの同期のスコープに含まれるオブジェクトを含む AD DS 組織単位を作成します。

   ```powershell
   New-ADOrganizationalUnit 'ToSync' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールから、以下を実行して、Windows 10 ドメイン参加済みのクライアント コンピューターのコンピューター オブジェクトを含む AD DS 組織単位を作成します。

   ```powershell
   New-ADOrganizationalUnit 'WVDClients' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. **[Administrator: Windows PowerShell ISE]** スクリプト ペインから、次を実行して、このラボで使用する Microsoft Entra テナントに同期される AD DS ユーザー アカウントを作成します (`<password>` プレースホルダーは、ランダムで複雑なパスワードに置き換えます)。

   > **注**: 使用したパスワードを必ず記録してください。 後でこのラボと以降のラボで必要になります。

   ```powershell
   $ouName = 'ToSync'
   $ouPath = "OU=$ouName,DC=adatum,DC=com"
   $adUserNamePrefix = 'aduser'
   $adUPNSuffix = 'adatum.com'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AdUser -Name $adUserNamePrefix$counter -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix$counter@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru
   } 

   $adUserNamePrefix = 'wvdadmin1'
   $adUPNSuffix = 'adatum.com'
   New-AdUser -Name $adUserNamePrefix -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru

   Get-ADGroup -Identity 'Domain Admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

   > **注**: このスクリプトは、**「aduser1** - **aduser9」** という名前の 9 つの非特権ユーザー アカウントと、**「wvdadmin1」** という名前の **ADATUM\\Domain Admins** グループのメンバーである 1 つの特権アカウントを作成します。

1. **[Administrator: Windows PowerShell ISE]** スクリプト ペインから、次を実行して、このラボで使用する Microsoft Entra テナントに同期される AD DS グループ オブジェクトを作成します。

   ```powershell
   New-ADGroup -Name 'az140-wvd-pooled' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-remote-app' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-personal' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-users' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-admins' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールから、以下を実行して、前の手順で作成したグループにメンバーを追加します。

   ```powershell
   Get-ADGroup -Identity 'az140-wvd-pooled' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4'
   Get-ADGroup -Identity 'az140-wvd-remote-app' | Add-AdGroupMember -Members 'aduser1','aduser5','aduser6'
   Get-ADGroup -Identity 'az140-wvd-personal' | Add-AdGroupMember -Members 'aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-users' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4','aduser5','aduser6','aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

#### タスク 2: AD DS UPN サフィックスを構成する

1. **az140-dc-vm11** への Bastion セッション内で、**[管理者: Windows PowerShell ISE]** スクリプト ペインで、次のように実行して、PowerShellGet モジュールの最新バージョンをインストールします (確認を求められたら **[はい]** を選択します)。

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールから、以下を実行して、最新バージョンの Az PowerShell モジュールをインストールします (確認を求められたら、**[すべてはい]** を選択します)。

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

   > **注**:Az モジュールのインストールからの出力が表示されるまで、3 から 5 分待つ必要がある場合があります。 また、出力が停止した**後**、さらに 5 分待つ必要がある場合があります。 これは正しい動作です。

1. **[管理者: Windows PowerShell ISE]** コンソールで、次を実行して Windows アカウント マネージャーを無効にします。

   ```powershell
   Update-AzConfig -EnableLoginByWam $false
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールから、以下を実行して、Azure サブスクリプションにサインインします。

   ```powershell
   Connect-AzAccount
   ```

1. プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つ Entra ID ユーザーアカウントの資格情報を入力します。
1. **[Administrator: Windows PowerShell ISE]** コンソールから、次を実行して、Azure サブスクリプションに関連付けられている Microsoft Entra テナントの ID プロパティを取得します。

   ```powershell
   $tenantId = (Get-AzContext).Tenant.Id
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールから、以下を実行して、最新バージョンの Azure AD PowerShell モジュールをインストールしてインポートします。

   ```powershell
   Install-Module -Name AzureAD -Force
   Import-Module -Name AzureAD
   ```

1. **[Administrator: Windows PowerShell ISE]** コンソールから、次を実行して、Microsoft Entra テナントに対して認証します。

   ```powershell
   Connect-AzureAD -TenantId $tenantId
   ```

1. プロンプトが表示されたら、このタスクの前半で使用したのと同じ資格情報 (このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウント) でサインインします。 
1. **[Administrator: Windows PowerShell ISE]** コンソールから、次を実行して、Azure サブスクリプションに関連付けられている Microsoft Entra テナントのプライマリ DNS ドメイン名を取得します。

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. **[Administrator: Windows PowerShell ISE]** コンソールから、次を実行して、Azure サブスクリプションに関連付けられている Microsoft Entra テナントのプライマリ DNS ドメイン名を、AD DS フォレストの UPN サフィックスのリストに追加します。

   ```powershell
   Get-ADForest|Set-ADForest -UPNSuffixes @{add="$aadDomainName"}
   ```

1. **[Administrator: Windows PowerShell ISE]** スクリプト ペインから、次を実行して、Azure サブスクリプションに関連付けられている Microsoft Entra テナントのプライマリ DNS ドメイン名を、AD DS ドメイン内のすべてのユーザーの UPN サフィックスとして割り当てます。

   ```powershell
   $domainUsers = Get-ADUser -Filter {UserPrincipalName -like '*adatum.com'} -Properties userPrincipalName -ResultSetSize $null
   $domainUsers | foreach {$newUpn = $_.UserPrincipalName.Replace('adatum.com',$aadDomainName); $_ | Set-ADUser -UserPrincipalName $newUpn}
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールから次を実行して、**adatum.com** UPN サフィックスを **Student** ドメイン ユーザーにもう一度割り当てます。

   ```powershell
   $domainAdminUser = Get-ADUser -Filter {sAMAccountName -eq 'Student'} -Properties userPrincipalName
   $domainAdminUser | Set-ADUser -UserPrincipalName 'student@adatum.com'
   ```

#### タスク 3: ディレクトリ同期の構成に使用する Microsoft Entra ユーザーを作成する

1. **az140-dc-vm11** への Bastion セッション内で、**[管理者: Windows PowerShell ISE]** スクリプト ペインから次を実行して、新しい Microsoft Entra ユーザーを作成します (`<password>` プレースホルダーをランダムで複雑なパスワードに置き換えます)。

   > **注**: 使用したパスワードを必ず記録してください。 **このラボでこの後、およびこのつづきのラボで必要になります。**.

   ```powershell
   $userName = 'aadsyncuser'
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName $userName -PasswordProfile $passwordProfile -MailNickName $userName -UserPrincipalName "$userName@$aadDomainName"
   ```

1. **[Administrator: Windows PowerShell ISE]** スクリプト ペインから、次を実行して、新しく作成された Microsoft Entra ユーザーにグローバル管理者ロールを割り当てます。 

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "$userName@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'} 
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

1. **[Administrator: Windows PowerShell ISE]** スクリプト ペインから、次を実行して、新しく作成された Microsoft Entra ユーザーのユーザー プリンシパル名を識別します。

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq '$userName'").UserPrincipalName
   ```

   > **注**:ユーザー プリンシパル名**および**パスワードを記録します。 これはこの演習で後から必要になります。 


#### タスク 4: Microsoft Entra Connect をインストールする

1. **az140-dc-vm11** への Bastion セッション内で、**[管理者: Windows PowerShell ISE]** スクリプト ペインから、以下を実行して、TLS 1.2 を有効にします。

   ```powershell
   New-Item 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   Write-Host 'TLS 1.2 has been enabled.'
   ```
   
1. **az140-dc-vm11** への Bastion セッション内で、Internet Explorer を起動して、[Microsoft Edge for Business ダウンロード ページ](https://www.microsoft.com/en-us/edge/business/download)に移動します。
1. [「Microsoft Edge for Business のダウンロード ページ」](https://www.microsoft.com/en-us/edge/business/download)から、最新の安定バージョンの Microsoft Edge をダウンロードし、インストールして起動し、既定の設定で構成します。
1. **az140-dc-vm11** への [リモート デスクトップ] セッション内で、Microsoft Edge を使用して、[Azure portal](https://portal.azure.com) に移動します。 プロンプトが表示されたら、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの Microsoft Entra 資格情報を使用してサインインします。
1. Azure portal で、Azure portal ページの上部にある **[リソース、サービス、ドキュメントを検索する]** テキスト ボックスを使用し、**[Microsoft Entra ID]** ブレードを検索してそこに移動したら、[Microsoft Entra テナント] ブレードのハブ メニューの **[管理]** セクションで、**[Microsoft Entra Connect]** を選択します。
1. **[Microsoft Entra Connect]** ブレードで、サービス メニューから **[Connect 同期]** リンクを選択したら、**[Microsoft Entra Connect のダウンロード]** リンクを選択します。 この操作を行うと、**Microsoft Entra Connect** のダウンロード ページが表示されている新しいブラウザー タブが自動的に開きます。
1. **Microsoft Entra Connect** のダウンロード ページで、**[ダウンロード]** を選択します。
1. **AzureADConnect.msi** インストーラーを実行するか保存するかを確認するメッセージが表示されたら、**[実行]** を選択します。 そうでない場合は、ダウンロードした後でファイルを開き、**Microsoft Azure Active Directory Connect** ウィザードを開始します。
1. **Microsoft Azure Active Directory Connect** ウィザードの **[Azure AD Connect へようこそ]** ページで、チェック ボックス **[ライセンス条項とプライバシーに関する通知に同意します]** をオンにして、**[続行]** を選択します。
1. **Microsoft Azure Active Directory Connect** ウィザードの **[簡単設定]** ページで、**[カスタマイズ]** オプションを選択します。
1. **[必要なコンポーネントをインストールする]** ページで、オプションの構成オプションをすべて選択解除したままにして、 **[インストール]** を選択します。
1. **[ユーザー サインイン]** ページで、**[パスワード ハッシュの同期]** のみを確実に有効にして、**[次へ]** を選択します。
1. **[Azure AD に接続]** ページで、前の演習で作成した **aadsyncuser** ユーザー アカウントの資格情報を使用して認証し、**[次へ]** を選択します。 

   > **注**: この演習で前に記録した **aadsyncuser** アカウントの userPrincipalName 属性を指定し、パスワードとしてこのラボで前に設定したパスワードを指定します。

1. **[ディレクトリの接続]** ページで、**adatum.com** フォレスト エントリの右側にある **[ディレクトリの追加]** ボタンを選択します。
1. **[AD フォレスト アカウント]** ウィンドウで、**[新しい AD アカウントを作成]** オプションが選択されていることを保証し、次の資格情報を指定して、**[OK]** を選択します。

   |設定|値|
   |---|---|
   |ユーザー名|**ADATUM\Student**|
   |パスワード|**Pa55w.rd1234**|

1. **[ディレクトリの接続]** ページに戻り、**adatum.com** エントリが構成済みディレクトリとして表示されていることを保証し、**[次へ]** を選択します
1. **[Azure AD サインインの構成]** ページで、"**UPN サフィックスが検証済みのドメインと一致しない場合、ユーザーはオンプレミスの資格情報を使用して Azure AD にサインインできなくなります**" という警告に注意して、チェック ボックス **[一部の UPN サフィックスが確認済みドメインに一致していなくても続行する]** をオンにして、**[次へ]** を選択します。

   > **注**: Microsoft Entra テナントには、**adatum.com** AD DS の UPN サフィックスの 1 つと一致する検証済みのカスタム DNS ドメインがないため、これは想定内のことです。

1. **[ドメインと OU のフィルタリング]** ページで、オプション **[選択したドメインと OU の同期]** を選択し、adatum.com を展開して、すべてのチェック ボックスをオフにし、**[ToSync]** OU のチェック ボックスのみをオンにして、**[次へ]** を選択します。
1. **[一意のユーザー識別]** ページで、既定の設定をそのまま使用して、**[次へ]** を選択します。
1. **[ユーザーおよびデバイスのフィルタリング]** ページで、既定の設定をそのまま使用して、**[次へ]** を選択します。
1. **[オプション機能]** ページで、既定の設定をそのまま使用して、**[次へ]** を選択します。
1. **[構成の準備完了]** ページで、**[構成が完了したら、同期プロセスを開始する]** チェック ボックスがオンになっていることを保証し、**[インストール]** を選択します。

   > **注**:インストールには約 5 分かかります。

1. **[構成が完了しました]** ページの情報を確認し、**[終了]** を選択して、**[Microsoft Azure Active Directory Connect]** ウィンドウを閉じます。
1. **az140-dc-vm11** への [リモート デスクトップ] セッション内において、Azure portal を表示している Microsoft Edge ウィンドウで、Adatum Lab Microsoft Entra テナントの **[ユーザー - すべてのユーザー]** ブレードに移動します。
1. **[ユーザー] \| [すべてのユーザー]** ブレードで、ユーザー オブジェクトのリストに、このラボで前に作成した AD DS ユーザー アカウントのリストが含まれており、**[オンプレミスの同期が有効]** 列に **[はい]** エントリが表示されていることに注目します。

   > **注**: AD DSユーザー アカウントが表示されるまで数分待ってから、ブラウザー ページを更新する必要がある場合があります。
