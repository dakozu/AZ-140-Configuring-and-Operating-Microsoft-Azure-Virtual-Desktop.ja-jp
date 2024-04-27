---
lab:
  title: 'ラボ: Azure Virtual Desktop のデプロイの準備をする (Microsoft Entra DS)'
  module: 'Module 1: Plan an AVD Architecture'
---

# ラボ: Azure Virtual Desktop のデプロイの準備をする (Microsoft Entra DS)
# 受講生用ラボ マニュアル

## ラボの依存関係

- Azure サブスクリプション
- Azure サブスクリプションに関連付けられた Microsoft Entra テナントの全体管理者ロールと、Azure サブスクリプションの所有者または共同作成者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウント

## 推定所要時間

150 分

>**注**: Microsoft Entra DS のプロビジョニングには、約 90 分の待機時間が必要です。

## ラボのシナリオ

Azure Active Directory Domain Services (Microsoft Entra DS) 環境での Azure Virtual Desktop のデプロイの準備が必要です

## 目標
  
このラボを完了すると、次のことができるようになります。

- Microsoft Entra DS ドメインを実装する
- Microsoft Entra DS ドメイン環境を構成する

## ラボ ファイル

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json

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

1. Azure portal の **Cloud Shell** の PowerShell セッションで、以下を実行して、**Microsoft.Compute** リソース プロバイダーを登録します (登録されていない場合)。

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   ```

1. Azure portal の **Cloud Shell** の PowerShell セッションで、以下を実行して、**Microsoft.Compute** リソース プロバイダーの登録ステータスを確認します。

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**注**: 状態が "**登録済み**" と表示されていることを確認します。 そうでない場合は、数分待ってからこの手順を繰り返します。

1. Azure portal の **Cloud Shell** の PowerShell セッションで、以下を実行して、Azure リージョンの名前がある PowerShell 変数を作成します (`<Azure_region>` プレースホルダーをこのラボで使用する `eastus` などの Azure リージョンの名前に置き換えます)。

   ```powershell
   $location = '<Azure_region>'
   ```

   > **注**: Azure リージョンの名前を識別するには、**Cloud Shell** の PowerShell プロンプトで `(Get-AzLocation).Location` を実行します。
   
1. Azure portal の **Cloud Shell** の PowerShell セッションで、以下を実行して、vCPU の現在の使用状況と **StandardDSv3Family** Azure VM の対応する制限を特定します。

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

1. 前の手順で実行したコマンドの出力を確認し、対象の Azure リージョンにある Azure VM の **Standard DSv3 Family** で、使用可能な vCPU が少なくとも **20** 個あることを確認します。 既にそうである場合は、次の演習に直接進みます。 それ以外の場合は、この演習の次のタスクに進みます。 

#### タスク 2: vCPU クォータの引き上げを要求する

1. Azure portal で、「**サブスクリプション**」を検索して選択し、**[サブスクリプション]** ウィンドウから、このラボで使用する予定の Azure サブスクリプションを表すエントリを選択します。
1. Azure portal のサブスクリプション ブレードの左側の垂直メニューの **[設定]** セクションで、**[使用量 + クォータ]** を選択します。 
1.  **[Azure Pass – スポンサー プラン | 使用量 + クォータ]**  ウィンドウの上部にある検索バーで、次のドロップダウン矢印を選択します。

   |**設定**|**Value**|
   |---|---|
   |**検索**|**Standard DSv3**|
   |**すべての場所**|**すべてクリア**し、*"自分の場所"* を調べます|
   |**リソース プロバイダー** | **Microsoft.Compute** |
   
1. 返された **Standard BS Family vCPUs** 項目で、鉛筆アイコンの **[編集]** を選択します。
1. **[クォータの詳細]** ウィンドウの **[新しい制限]** 列のテキスト ボックスに「**30**」と入力して、**[保存して続行]** を選びます。
1. クォータの要求が完了するまで待ちます。  しばらくすると、**[クォータの詳細]** ブレードに、要求が承認され、クォータが増加したことが示されます。 **[クォータの詳細]** ブレードを閉じます。

    >**注**: Azure リージョンの選択と現在の需要によっては、サポート要求の提出が必要になる場合があります。 サポート要求の作成プロセスに関する手順については、「[Azure サポート要求を作成する](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request)」を参照してください。

### 演習 1: Azure Active Directory Domain Services (AD DS) ドメインを実装する

この演習の主なタスクは次のとおりです。

1. Microsoft Entra DS ドメインの管理用に Microsoft Entra ユーザー アカウントを作成して構成する
1. Azure portal を使用して Microsoft Entra DS インスタンスをデプロイする
1. Microsoft Entra DS デプロイのネットワークと ID の設定を構成する

#### タスク 1: Microsoft Entra DS ドメインの管理用に Microsoft Entra ユーザー アカウントを作成して構成する

1. ラボ コンピューターで Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動して、このラボで使用するサブスクリプションの所有者ロールと、Azure サブスクリプションに関連付けられている Microsoft Entra テナントのグローバル管理者ロールを持つユーザー アカウントの資格情報を入力してサインインします。
1. Azure portal を表示している Web ブラウザーで、Azure AD テナントの [**概要**] ウィンドウに移動し、左側にある縦のメニューの [**管理**] セクションで、[**プロパティ**] をクリックします。
1. Microsoft Entra テナントの **[プロパティ]** ブレードの一番下で、**[セキュリティの既定値群の管理]** リンクを選択します。
1. **[セキュリティ既定値の有効化]** ウィンドウで、必要に応じて、**[いいえ]** を選択し、**[自分の組織では条件付きアクセスを使用している]** チェックボックスを選択して、**[保存]** を選択します。
1. Azure portal で、検索テキスト ボックスのすぐ右にあるツール バー アイコンを選択して **[Cloud Shell]** ペインを開きます。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、 **[PowerShell]** を選択します。 

   >**注**: **Cloud Shell** を初めて起動し、[**ストレージがマウントされていません**] というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。 

1. Cloud Shell ウィンドウから以下を実行して、Microsoft Entra テナントにサインインします。

   ```powershell
   Connect-AzureAD
   ```

1. Cloud Shell ウィンドウから以下を実行して、Azure サブスクリプションに関連付けられた Microsoft Entra テナントのプライマリ DNS ドメイン名を取得します。

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   $aadDomainName
   ```

1. Cloud Shell ウィンドウから以下を実行して、昇格された特権が付与される Microsoft Entra ユーザーを作成します (`<password>` プレースホルダーをランダムで複雑なパスワードに置き換えます)。

   > **注**: 使用したパスワードを必ず覚えておいてください。 後でこのラボと以降のラボで必要になります。

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'aadadmin1' -PasswordProfile $passwordProfile -MailNickName 'aadadmin1' -UserPrincipalName "aadadmin1@$aadDomainName"
   New-AzureADUser -AccountEnabled $true -DisplayName 'wvdaadmin1' -PasswordProfile $passwordProfile -MailNickName 'wvdaadmin1' -UserPrincipalName "wvdaadmin1@$aadDomainName"
   ```

1. Cloud Shell ウィンドウから以下を実行して、新しく作成された Microsoft Entra ユーザーの最初のユーザーにグローバル管理者ロールを割り当てます。

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "aadadmin1@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'}
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **注**: Azure AD PowerShell モジュールは、グローバル管理者ロールを社内管理者として参照します。

1. Cloud Shell ウィンドウで、次のコマンドを実行して、新しく作成された Microsoft Entra ユーザーのユーザー プリンシパル名を特定します。

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").UserPrincipalName
   ```

   > **注**: ユーザー プリンシパル名を記録します。 これはこの演習で後から必要になります。 

1. [Cloud Shell] ペインを閉じます。
1. Azure portal 内で、「**サブスクリプション**」を検索して選択し、**[サブスクリプション]** ウィンドウから、このラボで使用している Azure サブスクリプションを選択します。 
1. Azure サブスクリプションのプロパティを表示するウィンドウで、**[アクセス制御 (IAM)]**、**[追加]**、**[ロールの割り当ての追加]** の順に選択します。 
1. **[ロールの割り当ての追加]** ウィンドウで、**[所有者]** を選択し、**[次へ]** をクリックします
1. **[+ メンバーの選択]** ハイパーリンクをクリックします。
1. [**メンバーの選択**] ブレードで、[**aadadmin**] 項目を選択し、[**選択**] ボタンをクリックし、[**次へ**] をクリックします。
1. **[確認と割り当て]** ウィンドウで、**[確認と割り当て]** ボタンを選択します。

   > **注**: **aadadmin1 アカウント**を使用して Azure サブスクリプションを管理し、ラボの後半で Windows 10 Azure VM に参加した Microsoft Entra DS の対応する Microsoft Entra テナントを管理します。 


#### タスク 2: Azure portal を使用して Microsoft Entra DS インスタンスをデプロイする

1. ラボ コンピューターの Azure portal で、「**Microsoft Entra Domain Services**」を検索して選択し、[**Microsoft Entra Domain Services**] ブレードで [**+ 作成**] を選択します。 これにより、[**Microsoft Entra Domain Services の作成**] ブレードが開きます。
1. [** Microsoft Entra Domain Services の作成**] ブレードの [**基本**] タブで、次の設定を指定し、[**次へ**] を選択します (他の設定は既定値のままにします)。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|新しい **az140-11a-RG**の作成を選択します|
   |DNS ドメイン名|**adatum.com**|
   |リージョン|AVD デプロイをホストするリージョンの名前|
   |SKU|**Standard**|

   > **注**: これは技術的には必須ではありませんが、一般的には、既存の Azure またはオンプレミスの DNS 名前空間とは異なる Microsoft Entra DS ドメイン名を割り当てる必要があります。

1. [**Microsoft Entra Domain Services の作成**] ブレードの [**ネットワーク**] タブで、[**仮想ネットワーク**] ドロップダウン リストの横にある [**新規作成**] を選択します。
1. **[仮想ネットワークの作成]** ウィンドウで、次のように指定してから、**[OK]** を選択します。

   |設定|値|
   |---|---|
   |名前|**az140-aadds-vnet11a**|
   |アドレス範囲|**10.10.0.0/16**|
   |サブネット名|**aadds-Subnet**|
   |サブネット名|**10.10.0.0/24**|

1. **[仮想ネットワークの作成]** ウィンドウの **[ネットワーク]** タブに戻り、**[次へ]** を選択します (他の設定は既存の値のままにします)。
1. [**Microsoft Entra Domain Services の作成**] ブレードの [**管理**] タブで、既定の設定をそのまま使用して、[**次へ**] を選択します。
1. [**Microsoft Entra Domain Services の作成**] ブレードの [**同期**] タブで、[**すべて**] が選択されていることを確認し、[**次へ**] を選択します。
1. [**Microsoft Entra Domain Services の作成**] ブレードの [**セキュリティ設定**] タブで、既定の設定をそのまま使用して、[**次へ**] を選択します。
1. [**Microsoft Entra Domain Services の作成**] ブレードの [**タグ**] タブで、既定の設定をそのまま使用して、[次へ] を選択します
2. [**Microsoft Entra Domain Services の作成**] ブレードの [**確認と作成**] タブで、[**作成**] を選択します。 
3. Microsoft Entra DS ドメインの作成後に変更できない設定に関する通知を確認し、[**OK**] を選択します。

   >**注**: Microsoft Entra DS ドメインのプロビジョニング後に変更できない設定には、DNS 名、Azure サブスクリプション、リソース グループ、ドメイン コントローラーをホストする仮想ネットワークとサブネット、フォレストの種類が含まれます。

   > **注**: このデプロイが完了するまで待ってから、次の演習に進んでください。 これには 90 分ほどかかる場合があります。 

#### タスク 3: Microsoft Entra DS デプロイのネットワークと ID の設定を構成する

1. ラボ コンピューターの Azure portal で、「**Microsoft Entra Domain Services**」を検索して選択し、[**Microsoft Entra Domain Services**] ブレードから **adatum.com** エントリを選択して、新しくプロビジョニングされた Microsoft Entra DS インスタンスに移動します。 
1. Microsoft Entra DS インスタンスの [**adatum.com**] ブレードで、「**マネージド ドメインの構成の問題が検出されました**」で始まる警告をクリックします。 
1. [**adatum.com | 構成の診断**] ブレードで、[**実行**] をクリックします。
1. **[検証]** セクションで、**[DNS レコード]** ペインを展開し、**[修正]** をクリックします。
1. **[DNS レコード]** ウィンドウで、もう一度 **[修正]** をクリックします。
1. Microsoft Entra DS インスタンスの [**adatum.com**] ブレードに戻り、[**必要な構成手順**] セクションで、Microsoft Entra DS のパスワード ハッシュ同期に関する情報を確認します。 

   > **注**: Microsoft Entra DS ドメイン コンピューターとそのリソースにアクセスする必要がある既存のクラウド専用ユーザーは、パスワードを変更するか、リセットする必要があります。 これは、先ほどこのラボで作成した **aadadmin1** アカウントに適用されます。

1. ラボ コンピューターから、Azure portal の [**Cloud Shell**] ウィンドウで **PowerShell** セッションを開きます。
1. [Cloud Shell] ウィンドウの PowerShell セッションから、次を実行して、Microsoft Entra **aadadmin1** ユーザー アカウントの objectID 属性を識別します。

   ```powershell
   Connect-AzureAD
   $objectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、前の手順で特定した objectId である **aadadmin1** ユーザー アカウントのパスワードをリセットします (`<password>` プレースホルダーをランダムで複雑なパスワードに置き換えます)。

   > **注**: 使用したパスワードを必ず覚えておいてください。 後でこのラボと以降のラボで必要になります。

   ```powershell
   $password = ConvertTo-SecureString '<password>' -AsPlainText -Force
   Set-AzureADUserPassword -ObjectId $objectId -Password $password -ForceChangePasswordNextLogin $false
   ```

   > **注**: 実際のシナリオでは、通常、**-ForceChangePasswordNextLogin** の値を $true に設定します。 ここでは、ラボの手順を簡略化するために **$false** を選択しました。 

1. 前の 2 つの手順を繰り返して、**wvdaadmin1** ユーザー アカウントのパスワードをリセットします。


### 演習 2: Microsoft Entra DS ドメイン環境を構成する
  
この演習の主なタスクは次のとおりです。

1. Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする
1. Azure Bastion をデプロイする
1. Microsoft Entra DS ドメインの既定の構成を確認する
1. Microsoft Entra DS に同期する AD DS ユーザーとグループを作成する

#### タスク 1: Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする

1. ラボ コンピューターの Azure portal を表示している Web ブラウザーで、Cloud Shell ウィンドウで PowerShell セッションを開き、以下を実行して、前のタスクで作成した **az140-aadds-vnet11a** という名前の仮想ネットワークに **cl-Subnet** という名前のサブネットを追加します。

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.10.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. ラボ コンピューターで、**\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** パラメーター ファイルを見つけ、Visual Studio Code を使用してそのファイルを開きます。

1.  行 21 で、domainPassword パラメーターの値を見つけます。 このラボで先ほど **aadadmin1** ユーザー アカウントに対して設定したパスワードを使用するようにパラメーター ファイルの既存のパスワードを更新し、ファイルを **[保存]** します。

1. Azure portal の Cloud Shell ペインのツール バーで、**[ファイルのアップロード/ダウンロード]** アイコンを選択し、ドロップダウン メニューで **[アップロード]** を選択して、**\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json** と **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** のファイルを Cloud Shell のホーム ディレクトリにアップロードします。
1. Cloud Shell ウィンドウの PowerShell セッションから、次を実行して、Azure Virtual Desktop クライアントとして機能する Windows 10 を実行する Azure VM をデプロイし、Microsoft Entra DS ドメインに参加させます。

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11a.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11a.parameters.json
   ```

   > **注**: デプロイには約 10 分かかります。 デプロイが完了するまで待ってから、次のタスクに進みます。 


#### タスク 2: Azure Bastion をデプロイする 

> **注**: Azure Bastion を使用すると、この演習の前のタスクでデプロイしたパブリック エンドポイントを使用せずに Azure VM に接続できると同時に、オペレーティング システム レベルの資格情報を対象とするブルート フォース攻撃から保護することができます。

> **注**: ブラウザーでポップアップ機能が有効になっていることを確認します。

1. Azure portal が表示されているブラウザー ウィンドウで、別のタブを開き、そのブラウザー タブで [**Azure portal**](https://portal.azure.com) に移動します。
1. Azure portal で、検索テキスト ボックスのすぐ右にあるツール バー アイコンを選択して **[Cloud Shell]** ペインを開きます。
1. Cloud Shell ペインの PowerShell セッションから以下を実行して、この演習で先ほど作成した **az140-adds-vnet11** という名前の仮想ネットワークに **AzureBastionSubnet** という名前のサブネットを追加します。

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.10.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. [Cloud Shell] ペインを閉じます。
1. Azure portal で **[複数の要塞]** を検索して選択し、**[複数の要塞]** ブレードから **[+ 追加]** を選択します。
1. **[Bastion の作成]** ウィンドウの **[基本]** タブで、次の設定を指定して、**[確認および作成]** を選びます。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**az140-11a-RG**|
   |名前|**az140-11a-bastion**|
   |リージョン|この演習の前のタスクでリソースをデプロイしたリージョンと同じ Azure リージョン|
   |レベル|**Basic**|
   |仮想ネットワーク|**az140-aadds-vnet11a**|
   |Subnet|**AzureBastionSubnet (10.10.254.0/24)**|
   |パブリック IP アドレス|**新規作成**|
   |パブリック IP の名前|**az140-aadds-vnet11a-ip**|

1. **[Bastion の作成]** ウィンドウの **[確認と作成]** タブで、**[作成]** を選択します。

   > **注**: デプロイが完了するまで待ってから、この演習の次のタスクに進みます。 デプロイには約 5 分かかります。


#### タスク 3: Microsoft Entra DS ドメインの既定の構成を確認する

> **注**: 新しく Microsoft Entra DS に参加したコンピューターにサインインする前に、サインインに使用するユーザー アカウントを **AAD DC 管理者** Microsoft Entra グループに追加する必要があります。 この Microsoft Entra グループは、Microsoft Entra DS インスタンスをプロビジョニングした Azure サブスクリプションに関連付けられている Microsoft Entra テナントに自動的に作成されます。

> **注**: Microsoft Entra DS インスタンスをプロビジョニングするときに、このグループに既存の Microsoft Entra ユーザー アカウントを設定することもできます。

1. ラボ コンピューターの Azure portal の Cloud Shell ウィンドウで、次を実行して、**aadadmin1** Microsoft Entra ユーザー アカウントを **AAD DC 管理者** Microsoft Entra グループに追加します。

   ```powershell
   Connect-AzureAD
   $groupObjectId = (Get-AzureADGroup -Filter "DisplayName eq 'AAD DC Administrators'").ObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $groupObjectId -RefObjectId $userObjectId
   ```

1. [Cloud Shell] ペインを閉じます。
1. ラボ コンピューターの Azure portal で、「**仮想マシン**」を検索して選択し、**[仮想マシン]** ウィンドウで **[az140-cl-vm11a]** エントリを選択します。 これにより、[**az140-cl-vm11a**] ブレードが開きます。
1. **[az140-cl-vm11a]** ウィンドウで、**[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、**[az140-cl-vm11a]** の **[Bastion]** タブで次の資格情報を入力し、**[接続]** を選択します。
1. プロンプトが表示されたら、このラボで先ほど特定したプリンシパル名と、このラボでこのユーザー アカウントを作成するときに設定したパスワードを使用して、**aadadmin1** ユーザーとしてサインインします。
1. **az140-cl-vm11a** の Azure VM への Bastion 内で、**Windows PowerShell ISE** を管理者として起動し、[**管理者: Windows PowerShell ISE**] スクリプト ペインから次を実行して、Active Directory と DNS 関連のリモート サーバー 管理ツールをインストールします。

   ```powershell
   Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.Dns.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.ServerManager.Tools~~~~0.0.1.0 -Online
   ```

   > **注**: インストールが完了するのを待ってから、次の手順に進みます。 これには 2 分ほどかかる場合があります。

1. **az140-cl-vm11a** の Azure VM への Bastion 内で、**[スタート]** メニューの **Windows 管理ツール**フォルダーに移動して展開し、ツールの一覧から **[Active Directory ユーザーとコンピューター]** を開始します。 
1. **[Active Directory ユーザーとコンピューター]** コンソールで、**AADDC コンピューター**および **AADDC ユーザー**組織単位を含む既定の階層を確認します。 前者には **az140-cl-vm11a** コンピューター アカウントが含まれており、後者には、Microsoft Entra DS インスタンスのデプロイをホストする Azure サブスクリプションに関連付けられている Microsoft Entra テナントから同期されたユーザー アカウントが含まれていることに注意してください。 **AADDC ユーザー**組織単位には、同じ Microsoft Entra テナントから同期された AAD DC 管理istrators** グループとそのグループ メンバーシップも含まれます**。 このメンバーシップは、Microsoft Entra DS ドメイン内で直接変更することはできませんが、代わりに、Microsoft Entra DS テナント内で管理する必要があります。 変更は、Microsoft Entra DS ドメインでホストされているグループのレプリカと自動的に同期されます。 

   **ヒント:** **[Active Directory ユーザーとコンピューター]** に、ドメイン関連のコンテンツが一覧表示されない場合は、**[Active Directory ユーザーとコンピューター]** を右クリックし、**[ドメインの変更]** を選択し、ドメイン **[Adatum]** を選択します。

   > **注**: 現時点では、グループには **aadadmin1** ユーザー アカウントのみが含まれています。

1. **[Active Directory ユーザーとコンピューター]** コンソールの **[AADDC ユーザー]** OU で、ユーザー アカウント **[aadadmin1]** を選択し、その **[プロパティ]** ダイアログ ボックスを表示して、**[アカウント]** タブに切り替えて、ユーザー プリンシパル名のサフィックスがプライマリ Azure AD DNS ドメイン名と一致し、変更できないことを確認してください。 
1. **[Active Directory ユーザーとコンピューター]** コンソールで、**ドメイン コントローラー**の組織単位の内容を確認し、ランダムに生成された名前の 2 つの ドメイン コントローラーのコンピューター アカウントが含まれていることに注意してください。 

#### タスク 4: Microsoft Entra DS に同期される AD DS ユーザーとグループを作成する

1. **az140-cl-vm11a** Azure VM への Bastion 内で、Microsoft Edge を起動し、[Azure portal](https://portal.azure.com) に移動し、ユーザー アカウント **aadadmin1** のユーザー プリンシパル名と、パスワードとしてこのラボで先ほど設定したパスワードを指定してサインインします。
1. Azure portal で **Cloud Shell** を開きます。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、**PowerShell** を選択します。 

   >**注**: **aadadmin1** ユーザー アカウントを使用して **Cloud Shell** を初めて起動するため、Cloud Shell のホーム ディレクトリを構成する必要があります。 "ストレージがマウントされていません" というメッセージが表示されたら、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。**** 

1. [Cloud Shell] ペインの [PowerShell] セッションから、次を実行してサインインし、Microsoft Entra テナントに対する認証を行います。

   ```powershell
   Connect-AzureAD
   ```

1. [Cloud Shell] ペインの [PowerShell] セッション内で次を実行し、Azure サブスクリプションに関連付けられている Microsoft Entra テナントのプライマリ DNS ドメイン名を取得します。

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. [Cloud Shell] ペインの [PowerShell] セッションから次を実行して、今後のラボで使用する Microsoft Entra ユーザー アカウントを作成します (`<password>` プレースホルダーをランダムで複雑なパスワードに置き換えます)。

   > **注**: 使用したパスワードを必ず覚えておいてください。 後でこのラボと以降のラボで必要になります。

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   $aadUserNamePrefix = 'aaduser'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AzureADUser -AccountEnabled $true -DisplayName "$aadUserNamePrefix$counter" -PasswordProfile $passwordProfile -MailNickName "$aadUserNamePrefix$counter" -UserPrincipalName "$aadUserNamePrefix$counter@$aadDomainName"
   } 
   ```

1. [Cloud Shell] ペインの [PowerShell] セッションから次を実行して、**az140-wvd-aadmins** という名前の Microsoft Entra グループを作成して、**aadadmin1** および **wvdaadmin1** ユーザー アカウントに追加します。

   ```powershell
   $az140wvdaadmins = New-AzureADGroup -Description 'az140-wvd-aadmins' -DisplayName 'az140-wvd-aadmins' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-aadmins'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'wvdaadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   ```

1. [Cloud Shell] ペインで、前の手順を繰り返して、今後のラボで使用するユーザー用の Microsoft Entra グループを作成し、以前に作成した Microsoft Entra ユーザー アカウントに追加します。

   >**注**: 仮想マシン上のクリップボードのサイズには制限があるため、リストされているすべてのコマンドレットが正しくコピーされるとは限りません。 仮想マシンでメモ帳を開き、Lightning Bolt コントロールの一部である Type Text (Type Clipboard Text 構造) を使用して、すべてのコマンドレットをメモ帳にコピーします。 すべてのコマンドレットがメモ帳にコピーされていることを確認したら、それらをブロックで切り取って Cloud Shell に貼り付け、実行します。

   ```powershell
   $az140wvdausers = New-AzureADGroup -Description 'az140-wvd-ausers' -DisplayName 'az140-wvd-ausers' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-ausers'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId

   $az140wvdaremoteapp = New-AzureADGroup -Description "az140-wvd-aremote-app" -DisplayName "az140-wvd-aremote-app" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-aremote-app"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId

   $az140wvdapooled = New-AzureADGroup -Description "az140-wvd-apooled" -DisplayName "az140-wvd-apooled" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apooled"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId

   $az140wvdapersonal = New-AzureADGroup -Description "az140-wvd-apersonal" -DisplayName "az140-wvd-apersonal" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apersonal"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   ```

1. [Cloud Shell] ペインを閉じます。
1. **az140-cl-vm11a** の Azure VM への Bastion 内で、Azure portal を表示している [Microsoft Edge] ウィンドウで、**[Azure Active Directory]** ブレードを検索して選択し、Microsoft Entra テナント ブレードの左側の垂直方向のメニュー バーにある **[管理]** セクションで、**[ユーザー]** を選択し、**[ユーザー \| すべてのユーザー]** ブレードで、新しいユーザー アカウントが作成されたことを確認します。
1. Microsoft Entra テナント ブレードに戻り、左側の垂直方向のメニュー バーにある **[管理]** セクションで **[グループ]** を選択し、**[グループ \| すべてのグループ]** ブレードで、新しいグループ アカウントが作成されたことを確認します。
1. **Az140-cl-vm11a** の Azure VM への Bastion 内で、**Active Directory ユーザーとコンピューター** コンソールに切り替え、**Active Directory ユーザーとコンピューター** コンソールで **AADDC Users** OU に移動し、同じユーザー アカウントとグループ アカウントが含まれていることを確認します。

   >**注**: コンソールのビューを更新する必要がある場合があります。
