---
lab:
  title: 'ラボ:Azure Virtual Desktop (Azure AD DS) のデプロイの準備'
  module: 'Module 1: Plan a AVD Architecture'
---

# <a name="lab---prepare-for-deployment-of-azure-virtual-desktop-azure-ad-ds"></a>ラボ - Azure Virtual Desktop (Azure AD DS) のデプロイを準備する
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-dependencies"></a>ラボの依存関係

- Azure サブスクリプション
- Azure サブスクリプションに関連付けられている Azure AD テナントでグローバル管理者ロールになっていて、Azure サブスクリプションで所有者または共同作成者のロールになっている、Microsoft アカウントまたは Azure AD アカウント

## <a name="estimated-time"></a>推定所要時間

150 分

>**注**:Azure AD DS のプロビジョニングには約 90 分の待機時間がかかります。

## <a name="lab-scenario"></a>ラボのシナリオ

Azure Active Directory ドメイン サービス (Azure AD DS) 環境での Azure 仮想デスクトップのデプロイの準備をする必要があります

## <a name="objectives"></a>目標
  
このラボを完了すると、次のことができるようになります。

- Azure AD DS ドメインを実装する
- Azure AD DS ドメイン環境を構築する

## <a name="lab-files"></a>ラボ ファイル 

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json

## <a name="instructions"></a>手順

### <a name="exercise-0-increase-the-number-of-vcpu-quotas"></a>演習 0:vCPU クォータの数を増やす

この演習の主なタスクは次のとおりです。

1. 現在の vCPU 使用状況を識別する
1. vCPU クォータの増加を要求する

#### <a name="task-1-identify-current-vcpu-usage"></a>タスク 1:現在の vCPU 使用状況を識別する

1. ラボのコンピューターから Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者の役割を持つユーザーアカウントの認証情報を提供してサインインします。
1. Azure portal で、検索テキストボックスのすぐ右にあるツールバー アイコンを選択して **Cloud Shell** ペインを開きます。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、 **[PowerShell]** を選択します。 

   >**注**: **Cloud Shell** を初めて起動し、"**ストレージがマウントされていません**" というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。 

1. **Microsoft.Compute** リソース プロバイダーが登録されていない場合は、Azure portal の **Cloud Shell** の PowerShell で、次のコマンドを実行して登録します。

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   ```

1. Azure portal の **Cloud Shell** の PowerShell で、次のコマンドを実行して、**Microsoft.Compute** リソース プロバイダーの登録の状態を確認します。

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**注**:状態が **[登録済み]** と表示されていることを確認します。 そうでない場合は、数分待ってから、この手順を繰り返します。

1. Azure portal の **Cloud Shell** の PowerShell セッションで、次のコマンドを実行して、vCPU の現在の使用状況と、**StandardDSv3Family** および **StandardBSFamily** Azure VM の対応する制限を特定します (`<Azure_region>` プレースホルダーは、このラボで使うつもりの Azure リージョンの名前に置き換えます。例: `eastus`)。

   ```powershell
   $location = '<Azure_region>'
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardBSFamily'}
   ```

   > **注**:Azure リージョンの名前を識別するには、**Cloud Shell** の PowerShell プロンプトで `(Get-AzLocation).Location` を実行します。
   
1. 前の手順で実行したコマンドの出力を確認し、ターゲット Azure リージョンの Azure VM の **Standard DSv3 Family** と **StandardBSFamily** の両方に少なくとも **20** 個の使用可能な vCPU があることを確認します。 それがすでに当てはまる場合は、次の演習に直接進んでください。 それ以外の場合は、この演習の次のタスクに進みます。 

#### <a name="task-2-request-vcpu-quota-increase"></a>タスク 2:vCPU クォータの増加を要求する

1. Azure portal で、**[サブスクリプション]** を検索して選択し、 **[サブスクリプション]** ブレードから、このラボで使用する予定の Azure サブスクリプションを表すエントリを選択します。
1. Azure portal のサブスクリプション ブレードの左側の垂直メニューの **[設定]** セクションで、 **[使用量 + クォータ]** を選択します。 
1.  **[Azure Pass – スポンサー プラン | 使用量 + クォータ]**   ウィンドウの上部にある検索バーで、次のドロップダウン矢印を選択します。

|**設定**|**Value**|
|---|---|
|**Search**|**Standard BS**|
|**すべての場所**|**すべてクリア**し、 *"自分の場所"* を調べます|
|**リソース プロバイダー** | **Microsoft.Compute** |
   
1. 返された **Standard BS Family vCPUs** 項目で、鉛筆アイコンを選択し、**[編集]** を選択します。
1. **[クォータの詳細]** ウィンドウの **[新しい制限]** 列のテキスト ボックスに「**30**」と入力して、**[保存して続行]** を選びます。
1. クォータの要求が完了するまで待ちます。  しばらくすると、 **[クォータの詳細]** ブレードに要求が承認されてクォータが増えたことが示されます。 **[クォータの詳細]** ウィンドウを閉じます。
1. 検索の値を **Standard DSv3** に置き換え、クォータを増加して、上記の手順 3 から 6 を繰り返します。


### <a name="exercise-1-implement-an-azure-active-directory-domain-services-ad-ds-domain"></a>演習 1:Active Directory Domain Services (AD DS) ドメインを実装する

この演習の主なタスクは次のとおりです。

1. Azure AD DS ドメインを管理するためのAzure AD ユーザー アカウントを作成および構成する
1. Azure portal を使用して Azure AD DS インスタンスをデプロイする
1. Azure AD DS デプロイのネットワークと ID 設定を構成する

#### <a name="task-1-create-and-configure-an-azure-ad-user-account-for-administration-of-azure-ad-ds-domain"></a>タスク 1:Azure AD DS ドメインを管理するためのAzure AD ユーザー アカウントを作成および構成する

1. ラボのコンピューターから Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者のロールと、Azure サブスクリプションに関連付けられた Azure AD テナントのグローバル管理者のロールを持つユーザー アカウントの資格情報を提供して、サインインします。
1. Azure portal を表示している Web ブラウザーで、Azure AD テナントの **[概要]** ウィンドウに移動し、左側の垂直メニューの **[管理]** セクションで、**[プロパティ]** をクリックします。
1. Azure AD テナントの **[プロパティ]** ウィンドウの一番下で、**[セキュリティ詳細の管理]** リンクをクリックします。
1. **[セキュリティ既定値の有効化]** ブレードで、必要に応じて、 **[いいえ]** を選択し、 **[私の組織は条件付きアクセスを使用しています]** チェックボックスを選択して、 **[保存]** を選択します。
1. Azure portal で、検索テキストボックスのすぐ右にあるツールバー アイコンを選択して「**Cloud Shell**」ペインを開きます。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、 **[PowerShell]** を選択します。 

   >**注**: **Cloud Shell** を初めて起動し、"ストレージがマウントされていません" というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。**** 

1. Cloud Shell ペインから次のコマンドを実行して、Azure AD テナントににサインインします。

   ```powershell
   Connect-AzureAD
   ```

1. Cloud Shell ペインから、以下を実行して、Azure サブスクリプションに関連付けられている Azure AD テナントのプライマリ DNS ドメイン名を取得します。

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   $aadDomainName
   ```

1. Cloud Shell ペインから、以下を実行して、昇格された特権を付与する Azure AD ユーザーを作成します (`<password>` プレースホルダーをランダムで複雑なパスワードに置き換えます)。

   > **注**:使用したパスワードを必ず憶えておいてください。 後でこのラボと以降のラボで必要になります。

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'aadadmin1' -PasswordProfile $passwordProfile -MailNickName 'aadadmin1' -UserPrincipalName "aadadmin1@$aadDomainName"
   New-AzureADUser -AccountEnabled $true -DisplayName 'wvdaadmin1' -PasswordProfile $passwordProfile -MailNickName 'wvdaadmin1' -UserPrincipalName "wvdaadmin1@$aadDomainName"
   ```

1. Cloud Shell ペインから、次の操作を実行して、新しく作成された 最初の Azure AD ユーザーにグローバル管理者ロールを割り当てます。

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "aadadmin1@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'}
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **注**:Azure AD PowerShell モジュールは、グローバル管理者ロールを会社の管理者と呼びます。

1. Cloud Shell ウィンドウから、次の操作を実行して、新しく作成された Azure AD ユーザーのユーザー プリンシパル名を識別します。

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").UserPrincipalName
   ```

   > **注**:ユーザー プリンシパル名を記録します。 これはこの演習で後から必要になります。 

1. [Cloud Shell] ペインを閉じます。
1. Azure portal 内で、「**サブスクリプション**」を検索して選択し、**[サブスクリプション]** ウィンドウから、このラボで使用している Azure サブスクリプションを選択します。 
1. Azure サブスクリプションのプロパティを表示するウィンドウで、**[アクセス制御 (IAM)]**、**[追加]**、**[ロールの割り当ての追加]** の順に選択します。 
1. **[ロールの割り当ての追加]** ウィンドウで、**[所有者]** を選択し、**[次へ]** をクリックします
1. **[+ メンバーの選択]** ハイパーリンクをクリックします。
1. **[メンバーの選択]** ウィンドウで、**[aadadmin]** 項目を選択し、**[選択]** ボタンをクリックし、**[次へ]** をクリックします。
1. **[レビューと割り当て]** ウィンドウで、**[レビューと割り当て]** ボタンを選択します。

   > **注**:**aadadmin1** アカウントを使用して、Azure サブスクリプションと、ラボの後半で Windows 10 Azure VM に参加した Azure AD DS からの対応する Azure AD テナントを管理します。 


#### <a name="task-2-deploy-an-azure-ad-ds-instance-by-using-the-azure-portal"></a>タスク 2:Azure portal を使用して Azure AD DS インスタンスをデプロイする

1. ラボ コンピューターの Azure portal で、「**Azure AD Domain Services**」を検索して選択し、**[Azure AD Domain Services]** ウィンドウで **[+ 作成]** を選択します。 これにより、**[Azure AD Domain Services]** ウィンドウが開きます。
1. **[Azure AD Domain Services の作成]** ウィンドウの **[基本]** タブで、次の設定を指定し、**[次へ]** を選択します (他の設定は既存の値のままにします)。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|新しい **az140-11a-RG**の作成を選択します|
   |DNS ドメイン名|**adatum.com**|
   |Region|AVD デプロイをホストするリージョンの名前|
   |SKU|**Standard**|
   |フォレストの種類|**User**|

   > **注**:これは技術的には必須ではありませんが、一般に、既存の Azure またはオンプレミスの DNS 名前空間とは異なる Azure AD DS ドメイン名を割り当てる必要があります。

1. **[Azure AD Domain Services の作成]** ウィンドウの **[ネットワーク]** タブで、**[仮想ネットワーク]** ドロップダウン リストの横にある **[新規作成]** を選択します。
1. **[仮想ネットワークの作成]** ウィンドウで、次のように指定してから、**[OK]** を選択します。

   |設定|値|
   |---|---|
   |名前|**az140-aadds-vnet11a**|
   |アドレス範囲|**10.10.0.0/16**|
   |サブネット名|**aadds-Subnet**|
   |サブネット名|**10.10.0.0/24**|

1. **[仮想ネットワークの作成]** ウィンドウの **[ネットワーク]** タブに戻り、**[次へ]** を選択します (他の設定は既存の値のままにします)。
1. **[Azure AD Domain Services の作成]** ウィンドウの **[管理]** タブで、既定の設定を受け入れ、**[次へ]** を選択します。
1. **[Azure AD Domain Services の作成]** ウィンドウの **[同期]** タブで、**[すべて]** が選択されていることを確認し、**[次へ]** を選択します。
1. **[Azure AD Domain Services の作成]** ウィンドウの **[セキュリティ設定]** タブで、既定の設定を受け入れ、**[次へ]** を選択します。
1. **[Azure AD Domain Services の作成]** ウィンドウの **[管理]** タブで、既定の設定を受け入れ、[次へ] を選択します
2. **[Azure AD Domain Services の作成]** ウィンドウの **[確認および作成]** タブで、**[作成]** を選択します。 
3. Azure AD DS ドメインの作成後に変更できない設定に関する通知を確認し、**[OK]** を選択します。

   >**注**:Azure AD DS ドメインのプロビジョニング後に変更できない設定には、DNS 名、Azure サブスクリプション、リソース グループ、ドメイン コントローラーをホストする仮想ネットワークとサブネット、およびフォレストの種類が含まれます。

   > **注**: このデプロイが完了するまで待ってから、次の演習に進んでください。 これには 90 分ほどかかる場合があります。 

#### <a name="task-3-configure-the-network-and-identity-settings-of-the-azure-ad-ds-deployment"></a>タスク 3:Azure AD DS デプロイのネットワークと ID 設定を構成する

1. ラボ コンピューターの Azure portal で、「**Azure AD Domain Services**」を検索して選択し、**[Azure AD Domain Services]** ウィンドウから **[adatum.com]** エントリを選択して、新しくプロビジョニングされた Azure AD DS インスタンスに移動します。 
1. Azure AD DS インスタンスの **adatum.com** ブレードで、**管理するドメインの構成の問題が検出された**ことを示す警告をクリックします。 
1. **[adatum.com | 構成の診断 (プレビュー)]** ウィンドウで、**[実行]** をクリックします。
1. **[検証]** セクションで、**[DNS レコード]** ペインを展開し、**[修正]** をクリックします。
1. **[DNS レコード]** ウィンドウで、もう一度 **[修正]** をクリックします。
1. Azure AD DS インスタンスの **[adatum.com]** ウィンドウに戻り、**[必要な構成手順]** セクションで、Azure AD DS のパスワード ハッシュ同期に関する情報を確認します。 

   > **注**:Azure AD DS ドメイン コンピューターとそのリソースにアクセスできる必要がある既存のクラウドのみのユーザーは、パスワードを変更するか、パスワードをリセットする必要があります。 これは、このラボの前半で作成した **aadadmin1** アカウントに適用されます。

1. ラボ コンピューターの Azure portal で、**CloudShell** ペインで **PowerShell** セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、Azure AD **aadadmin1** ユーザー アカウントの objectID 属性を識別します。

   ```powershell
   Connect-AzureAD
   $objectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、前の手順で特定した objectId である **aadadmin1** ユーザー アカウントのパスワードをリセットします (`<password>` プレースホルダーをランダムで複雑なパスワードに置き換えます)。

   > **注**:使用したパスワードを必ず憶えておいてください。 後でこのラボと以降のラボで必要になります。

   ```powershell
   $password = ConvertTo-SecureString '<password>' -AsPlainText -Force
   Set-AzureADUserPassword -ObjectId $objectId -Password $password -ForceChangePasswordNextLogin $false
   ```

   > **注**:実際のシナリオでは、通常、 **-ForceChangePasswordNextLogin**の値を $true に設定します。 この場合、ラボの手順を簡素化するために **$false** を選択しました。 

1. 前述の 2 つの手順を繰り返して、**wvdaadmin1** ユーザー アカウントのパスワードをリセットします。


### <a name="exercise-2-configure-the-azure-ad-ds-domain-environment"></a>演習 2:Azure AD DS ドメイン環境を構築する
  
この演習の主なタスクは次のとおりです。

1. Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする
1. Azure Bastion をデプロイする
1. Azure AD DS ドメインの既定の構成を確認する
1. Azure AD DS と同期する AD DS ユーザーとグループを作成する

#### <a name="task-1-deploy-an-azure-vm-running-windows-10-by-using-an-azure-resource-manager-quickstart-template"></a>タスク 1:Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする

1. ラボ コンピューターから、Azure portal で、Cloud Shell ペインの PowerShell セッションから、以下を実行して、前のタスクで作成した **az140-aadds-vnet11a** という名前の仮想ネットワークに **cl-Subnet** という名前のサブネットを追加します。

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.10.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. ラボ コンピューターで、 **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** パラメーター ファイルを見つけ、Visual Studio Code を使用してそのファイルを開きます。

1.  行 21 で、domainPassword パラメーターの値を見つけます。 このラボで先ほど **aadadmin1** ユーザー アカウントに対して設定したパスワードを使用するようにパラメーター ファイルの既存のパスワードを更新し、ファイルを **[保存]** します。

1. Azure portal の Cloud Shell ペインのツール バーで、**[ファイルのアップロード/ダウンロード]** アイコンを選択し、ドロップダウン メニューで **[アップロード]** を選択して、**\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json** と **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** のファイルを Cloud Shell のホーム ディレクトリにアップロードします。
1. Cloud Shell ペインの PowerShell セッションから、次のコマンドを実行して、Azure Virtual Desktop クライアントとして機能する Windows 10 を実行している Azure VM を展開し、Azure AD DS ドメインに参加させます。

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

   > **注**: デプロイには約 10 分かかります。 次のタスクを進める前に、展開が完了するのを待ちます。 


#### <a name="task-2-deploy-azure-bastion"></a>タスク 2: Azure Bastion をデプロイする 

> **注**: Azure Bastion を使用すると、この演習の前のタスクでデプロイしたパブリック エンドポイントを使用せずに Azure VM に接続できると同時に、オペレーティング システム レベルの資格情報を対象とするブルート フォース攻撃から保護することができます。

> **注**:ブラウザーでポップアップ機能が有効になっていることを確認します。

1. Azure portal が表示されているブラウザー ウィンドウで、別のタブを開き、そのブラウザー タブで Azure portal に移動します。
1. Azure portal で、検索テキストボックスのすぐ右にあるツール バー アイコンを選択して、**Cloud Shell**ペインを開きます。
1. Cloud Shell ペインの PowerShell セッションから次のように実行して、この演習で先ほど作成した仮想ネットワーク **az140-adds-vnet11** に **AzureBastionSubnet** という名前のサブネットを追加します。

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
1. Azure portal で **[Bastions]** を検索して選び、 **[Bastions]** ブレードから **[+ 追加]** を選びます。
1. **[Bastion の作成]** ブレードの **[基本]** タブで、次の設定を指定して、 **[確認と作成]** を選びます。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**az140-11a-RG**|
   |名前|**az140-11a-bastion**|
   |Region|この演習の前のタスクでリソースをデプロイしたリージョンと同じ Azure リージョン|
   |レベル|**Basic**|
   |仮想ネットワーク|**az140-aadds-vnet11a**|
   |Subnet|**AzureBastionSubnet (10.10.254.0/24)**|
   |パブリック IP アドレス|**新規作成**|
   |パブリック IP の名前|**az140-aadds-vnet11a-ip**|

1. **[Bastion の作成]** ウィンドウの **[確認と作成]** タブで、 **[作成]** を選択します。

   > **注**:デプロイが完了するのを待ってから、この演習の次のタスクに進みます。 デプロイには約 5 分かかります。


#### <a name="task-3-review-the-default-configuration-of-the-azure-ad-ds-domain"></a>タスク 3:Azure AD DS ドメインの既定の構成を確認する

> **注**:新しく AzureAD DS に参加したコンピューターにサインインする前に、サインインする予定のユーザーアカウントを **AAD DC 管理者** Azure AD グループに追加する必要があります。 この Azure AD グループは、Azure AD DS インスタンスをプロビジョニングした Azure サブスクリプションに関連付けられた Azure AD テナントに自動的に作成されます。

> **注**:Azure AD DS インスタンスをプロビジョニングするときに、このグループに既存の Azure AD ユーザー アカウントを追加するオプションがあります。

1. ラボ コンピューターの Azure portalの Cloud Shell ペインで、次のコマンドを実行して、**aadadmin1** Azure AD ユーザー アカウントを **AAD DC 管理者** Azure AD グループに追加します。

   ```powershell
   Connect-AzureAD
   $groupObjectId = (Get-AzureADGroup -Filter "DisplayName eq 'AAD DC Administrators'").ObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $groupObjectId -RefObjectId $userObjectId
   ```

1 Cloud Shell ペインを閉じます。
1. ラボ コンピューターの Azure portal で、「**仮想マシン**」を検索して選択し、**[仮想マシン]** ウィンドウで **[az140-cl-vm11a]** エントリを選択します。 これにより、**az140-cl-vm11a** ブレードが開きます。
1. **[az140-cl-vm11a]** ウィンドウで、**[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、**[az140-cl-vm11a]** の **[Bastion]** タブで次の資格情報を入力し、**[接続]** を選択します。
1. プロンプトが表示されたら、このラボで先ほど特定したプリンシパル名と、このラボでこのユーザー アカウントを作成するときに設定したパスワードを使用して、**aadadmin1** ユーザーとしてサインインします。
1. **az140-cl-vm11a** Azure VM へのリモート デスクトップ内で、管理者として **Windows PowerShell ISE** を起動し、**管理者: Windows PowerShell ISE** スクリプト」ペインで以下を実行して、Active Directory と DNS 関連のリモート サーバー管理ツールをインストールします。

   ```powershell
   Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.Dns.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.ServerManager.Tools~~~~0.0.1.0 -Online
   ```

   > **注**:インストールが完了するのを待ってから、次の手順に進みます。 これには 2 分ほどかかる場合があります。

1. **az140-cl-vm11a** Azure VM へのリモート デスクトップ内で、**[スタート]** メニューの **[Windows 管理ツール]** フォルダーに移動して展開し、**[Active Directory ユーザーとコンピューター]** を選択します。 
1. **Active Directory ユーザーとコンピューター** コンソールで、**AADDC コンピューター**と **AADDC ユーザー**の組織単位を含むデフォルトの階層を確認します。 前者には **az140-cl-vm11a** コンピューター アカウントが含まれ、後者には Azure ADDS インスタンスの展開をホストする Azure サブスクリプションに関連付けられた Azure AD テナントから同期されたユーザー アカウントが含まれることに注意してください。 **AADDC ユーザー**組織単位には、同じ Azure AD テナントから同期された **AADDC 管理者**グループとそのグループ メンバーシップも含まれます。 このメンバーシップは、Azure AD DS ドメイン内で直接変更することはできませんが、代わりに、Azure AD DS テナント内で管理する必要があります。 変更はすべて、Azure AD DS ドメインでホストされているグループのレプリカと自動的に同期されます。 

**ヒント:** **[Active Directory ユーザーとコンピューター]** に、ドメイン関連のコンテンツが一覧表示されない場合は、**[Active Directory ユーザーとコンピューター]** を右クリックし、**[ドメインの変更]** を選択し、ドメイン **[Adatum]** を選択します。

   > **注**:現在、グループには **aadadmin1** ユーザー アカウントのみが含まれています。

1. **[Active Directory ユーザーとコンピューター]** コンソールの **[AADDC ユーザー]** OU で、ユーザー アカウント **[aadadmin1]** を選択し、その **[プロパティ]** ダイアログ ボックスを表示し、**[アカウント]** タブに切り替えて、ユーザー プリンシパル名のサフィックスがプライマリ Azure AD DNS ドメイン名と一致し、変更できないことを確認してください。 
1. **Active Directory ユーザーとコンピューター** コンソールで、**ドメイン コントローラー**組織単位の内容を確認し、ランダムに生成された名前を持つ 2 つのドメイン コントローラーのコンピューター アカウントが含まれていることに注意してください。 

#### <a name="task-4-create-ad-ds-users-and-groups-that-will-be-synchronized-to-azure-ad-ds"></a>タスク 4:Azure AD DS と同期する AD DS ユーザーとグループを作成する

1. **az140-cl-vm11a** Azure VM へのリモート デスクトップ内で、Microsoft Edge を起動し、[Azure portal](https://portal.azure.com) に移動し、ユーザー アカウント **aadadmin1** のユーザー プリンシパル名と、パスワードとしてこのラボで先ほど設定したパスワードを指定してサインインします。
1. Azure portal で **Cloud Shell** を開きます。
1. **Bash** や **PowerShell** のどちらかを選択するプロンプトが表示されたら、**PowerShell** を選択します。 

   >**注**:**aadadmin1** ユーザー アカウントを使用して **Cloud Shell** を起動するのはこれが初めてなので、Cloud Shell のホームディレクトリを構成する必要があります。 "ストレージがマウントされていません" というメッセージが表示されたら、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。**** 

1. Cloud Shell ペインの PowerShell セッションから、次のコマンドを実行してサインインし、Azure AD テナントに認証します。

   ```powershell
   Connect-AzureAD
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、Azure サブスクリプションに関連付けられている Azure AD テナントのプライマリ DNS ドメイン名を取得します。

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Cloud Shell ペインの PowerShell セッションから以下を実行して、今後のラボで使用する Azure AD ユーザー アカウントを作成します (`<password>` プレースホルダーをランダムで複雑なパスワードに置き換えます)。

   > **注**:使用したパスワードを必ず憶えておいてください。 後でこのラボと以降のラボで必要になります。

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

1. Cloud Shell ペインの PowerShell セッションから以下を実行して、**az140-wvd-aadmins** という名前の Azure AD グループを作成し、それに **aadadmin1** および **wvdaadmin1** ユーザー アカウントを追加します。

   ```powershell
   $az140wvdaadmins = New-AzureADGroup -Description 'az140-wvd-aadmins' -DisplayName 'az140-wvd-aadmins' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-aadmins'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'wvdaadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   ```

1. Cloud Shell ペインから、前の手順を繰り返して、今後のラボで使用するユーザー用の Azure AD グループを作成し、以前に作成した Azure AD ユーザー アカウントを追加します。

>**注**:注: 仮想マシン上のクリップボードのサイズには制限があるため、リストされているすべてのコマンドレットが正しくコピーされるとは限りません。 仮想マシンでメモ帳を開き、Lightning Bolt コントロールの一部である Type Text (Type Clipboard Text 構造) を使用して、すべてのコマンドレットをメモ帳にコピーします。 すべてのコマンドレットがメモ帳にコピーされていることを確認したら、それらをブロックで切り取って Cloud Shell に貼り付け、実行します。

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
1. **az140-cl-vm11a** Azure VM へのリモート デスクトップ内で、Azure portal を表示している Microsoft Edge ウィンドウで、「**Azure Active Directory** ウィンドウ」を検索して選択し、Azure AD テナント ウィンドウの左側の垂直メニュー バーにある **[管理]** セクションで、**[ユーザー]** を選択し、**[ユーザー \| すべてのユーザー]** ウィンドウで、新しいユーザー アカウントが作成されたことを確認します。
1. Azure AD テナント ウィンドウに戻り、左側の垂直メニュー バーにある **[管理]** セクションで **[グループ]** を選択し、**[グループ \| すべてのグループ**] ウィンドウで、新しいグループ アカウントが作成されたことを確認します。
1. リモート デスクトップ内で **az140-cl-vm11a** Azure VM に移動し、**Activ eDirectory ユーザーとコンピューター** コンソールに切り替えます。**Active Directory ユーザーとコンピューター** コンソールで、**AADDC ユーザー** OU に移動し、同じユーザー アカウントとグループ アカウントが含まれていることを確認します。

   >**注**:コンソールのビューを更新する必要がある場合があります。
