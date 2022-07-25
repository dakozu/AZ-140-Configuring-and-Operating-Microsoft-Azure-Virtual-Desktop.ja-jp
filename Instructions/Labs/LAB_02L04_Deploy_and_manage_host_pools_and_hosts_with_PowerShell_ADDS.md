---
lab:
  title: 'ラボ:PowerShell を使用してホスト プールとホストをデプロイおよび管理する (AD DS)'
  module: 'Module 2: Implement a WVD Infrastructure'
---

# <a name="lab---deploy-and-manage-host-pools-and-hosts-by-using-powershell"></a>ラボ - PowerShell を使用してホスト プールとホストをデプロイおよび管理する
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-dependencies"></a>ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプション内で所有者または共同作成者のロールを持つ Microsoft アカウントまたは Azure AD アカウント、この Azure サブスクリプションに関連付けられた Azure AD テナント内でグローバル管理者ロールを持つMicrosoft アカウントまたは Azure AD アカウント。
- 実施するラボ - **Azure Virtual Desktop (AD DS) のデプロイを準備する**

## <a name="estimated-time"></a>推定所要時間

約 60 分

## <a name="lab-scenario"></a>ラボのシナリオ

Active Directory ドメイン サービス (AD DS) 環境で PowerShell を使用して、Azure Virtual Desktop ホスト プールとホストのデプロイを自動化する必要があります。

## <a name="objectives"></a>目標
  
このラボを完了すると、次のことができるようになります。

- PowerShell を使用して、Azure Virtual Desktop ホスト プールとホストをデプロイする
- PowerShell を使用して、Azure Virtual Desktop ホスト プールにホストを追加する

## <a name="lab-files"></a>ラボ ファイル 

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json

## <a name="instructions"></a>手順

### <a name="exercise-1-implement-azure-virtual-desktop-host-pools-and-session-hosts-by-using-powershell"></a>演習 1:PowerShell を使用して、Azure Virtual Desktop ホスト プールとセッション ホストを実装する
  
この演習の主なタスクは次のとおりです。

1. PowerShell を使用して、Azure Virtual Desktop ホスト プールのデプロイを準備する
1. PowerShell を使用して Azure Virtual Desktop ホスト プールを作成する
1. PowerShell を使用して Windows 10 Enterprise を実行する Azure VM のテンプレートベースのデプロイを実行する
1. PowerShell を使用して Azure Virtual Desktop ホスト プールにセッション ホストとして Windows 10 Enterprise を実行する Azure VM を追加する
1. Azure Virtual Desktop セッション ホストのデプロイを確認する

#### <a name="task-1-prepare-for-deployment-of-azure-virtual-desktop-host-pool-by-using-powershell"></a>タスク 1:PowerShell を使用して、Azure Virtual Desktop ホスト プールのデプロイを準備する

1. ラボのコンピューターから Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者の役割を持つユーザーアカウントの認証情報を提供してサインインします。
1. Azure portal で、「**仮想マシン**」を検索して選択し、 **[Virtual Machines]** ブレードで、 **[az140-vm11]** を選択します。
1. **[az140-dc-vm11]** ウィンドウで **[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、 **[az140-dc-vm11 \| 接続]** ウィンドウの **[Bastion]** タブで **[Bastion を使用する]** を選択します。
1. プロンプトが表示されたら、次の資格情報を入力し、 **[接続]** を選択します。

   |設定|値|
   |---|---|
   |[ユーザー名]|**学生**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、**Windows PowerShell ISE** を管理者として起動します。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、Azure Virtual Desktop プール ホスト セッションのコンピューター オブジェクトをホストする **WVDInfra** という名前の組織単位の識別名を識別します。

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから次のように実行して、Azure Virtual Desktop を AD DS ドメイン ( **student@adatum.com** ) に参加させるために使用する **ADATUM\\Student** アカウントの UPN サフィックスを特定します。

   ```powershell
   (Get-ADUser -Filter {sAMAccountName -eq 'student'} -Properties userPrincipalName).userPrincipalName
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから次を実行して、DesktopVirtualization PowerShell モジュールをインストールします (プロンプトが表示されたら、 **[すべてはい]** をクリックします)。

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -Force
   ```

   > **注**:使用中の既存の PowerShell モジュールに関する警告は無視してください。

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Microsoft Edge を起動して、[Azure portal](https://portal.azure.com) に移動します。 プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を使用してサインインします。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal の Azure portal ページの上部にある **[リソース、サービス、およびドキュメントの検索]** テキスト ボックスを使用して、「**仮想ネットワーク**」と検索して移動し、 **[仮想ネットワーク]** ブレードで **[az140-adds-vnet11]** を選択します。 
1. **[az140-adds-vnet11]** ブレードで **[サブネット]** を選択し、 **[サブネット]** ブレードで **[+ サブネット]** を選択し、 **[サブネットの追加]** ブレードで次の設定を指定し (他のすべての設定は既定値のままにします)、 **[保存]** をクリックします。

   |設定|値|
   |---|---|
   |名前|**hp3-Subnet**|
   |サブネットのアドレス範囲|**10.0.3.0/24**|

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal で、Azure portal ページの上部にある **[リソース、サービス、およびドキュメントの検索]** テキストボックスを使用して、**ネットワーク セキュリティ グループ**を検索して移動し、 **[ネットワーク セキュリティ グループ]** ブレードで、**az140-11-RG** リソース グループでセキュリティ グループを選択します。
1. ネットワーク セキュリティ グループ ブレードの左側の垂直メニューの **[設定]** セクションで、 **[プロパティ]** をクリックします。
1. **[プロパティ]** ブレードで、 **[リソース ID]** テキストボックスの右側にある **[クリップボードにコピー]** アイコンをクリックします。 

   > **注**:値は、`/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg` のような形式になります。ただし、サブスクリプション ID は異なります。 次のタスクで必要になるので、記録します。

#### <a name="task-2-create-a-azure-virtual-desktop-host-pool-by-using-powershell"></a>タスク 2:PowerShell を使用して Azure Virtual Desktop ホスト プールを作成する

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから次を実行して、Azure サブスクリプションにサインインします。

   ```powershell
   Connect-AzAccount
   ```

1. プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を入力します。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから次を実行して、Azure 仮想ネットワーク **az140-adds-vnet11** をホストしている Azure リージョンを特定します。

   ```powershell
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから次を実行して、ホスト プールとそのリソースをホストするリソース グループを作成します。

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、空のホストプールを作成します。

   ```powershell
   $hostPoolName = 'az140-24-hp3'
   $workspaceName = 'az140-24-ws1'
   $dagAppGroupName = "$hostPoolName-DAG"
   New-AzWvdHostPool -ResourceGroupName $resourceGroupName -Name $hostPoolName -WorkspaceName $workspaceName -HostPoolType Pooled -LoadBalancerType BreadthFirst -Location $location -DesktopAppGroupName $dagAppGroupName -PreferredAppGroupType Desktop 
   ```

   > **注**:**New-AzWvdHostPool** コマンドレットを使用すると、ホスト プール、ワークスペース、およびデスクトップ アプリ グループを作成したり、デスクトップ アプリ グループをワークスペースに登録したりできます。 新しいワークスペースを作成するか、既存のワークスペースを使用するかを選択できます。

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから次を実行して、**az140-wvd-pooled** という名前の Azure AD グループの objectID 属性を取得します。

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-pooled').Id
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから次を実行して、**az140-wvd-pooled** という名前の Azure AD グループを、新しく作成されたホスト プールの既定のデスクトップ アプリ グループに割り当てます。

   ```powershell
   $roleDefinitionName = 'Desktop Virtualization User'
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName $roleDefinitionName -ResourceName $dagAppGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

#### <a name="task-3-perform-a-template-based-deployment-of-an-azure-vm-running-windows-10-enterprise-by-using-powershell"></a>タスク 3:PowerShell を使用して Windows 10 Enterprise を実行する Azure VM のテンプレートベースのデプロイを実行する

1. 自分のラボ コンピューターから、デプロイされたストレージ アカウントに移動します。 [ファイル共有] ブレードで、**az140-22-profiles** ファイル共有を選択します。

1. **[アップロード]** を選択して、両方のラボ ファイル **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json** と **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json** をファイル共有にアップロードします。

1. **az140-dc-vm11** への Bastion セッション内で、エクスプローラーを開き、前に構成した Z:、またはファイル共有への接続に割り当てたドライブ文字に移動します。 アップロードしたデプロイ ファイルを **C:\AllFiles\Labs\02** にコピーします。

1. **az140-dc-vm11** への Bastion セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから次のように実行して、前のタスクで作成したホストプールで Azure Virtual Desktop セッション ホストとして機能する Windows 10 Enterprise (マルチセッション) を実行している Azure VM をデプロイします。

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab24hp3Deployment `
     -TemplateFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.json `
     -TemplateParameterFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.parameters.json
   ```

   > **注**: デプロイが完了するまで待ってから、次のタスクに進んでください。 これには 5 分ほどかかる場合があります。 

   > **注**:デプロイでは、Azure Resource Manager テンプレートを使用して Azure VM をプロビジョニングし、オペレーティング システムを **adatum.com** AD DS ドメインに自動的に参加させる VM 拡張機能を適用します。

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソーから次のように実行して、3 番目のセッション ホストが **adatum.com** AD DS ドメインに正常に参加したことを確認します。

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-24-p3-0$'"
   ```

#### <a name="task-4-add-an-azure-vm-running-windows-10-enterprise-as-a-host-to-the-azure-virtual-desktop-host-pool-by-using-powershell"></a>タスク 4:PowerShell を使用して Azure Virtual Desktop ホスト プールにホストとして Windows 10 Enterprise を実行する Azure VM を追加する

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal を表示している Web ブラウザーの画面で、**仮想ネットワーク**を検索して選択し、 **[仮想ネットワーク]** ブレードで、仮想マシンの一覧で **az140-24-p3-0** を選択します。
1. **[az140-24-p3-0]** のブレードで **[接続]** を選び、ドロップダウン メニューで **[RDP]** を選び、 **[az140-24-p3-0] \| [接続]** ブレードの **[RDP]** タブの **[IP アドレス]** ドロップダウン リストで、 **[プライベート IP アドレス (10.0.3.4)]** エントリを選択し、次に **[RDP ファイルのダウンロード]** を選びます。
1. プロンプトが表示されたら、次の認証情報を入力します。

   |設定|値|
   |---|---|
   |[ユーザー名]|**ADATUM\\Student**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-24-p3-0** へのリモート デスクトップ セッション内で、**Windows PowerShell ISE** を管理者として起動します。
1. **az140-24-p3-0** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから次のように実行して、新しくデプロイされた Azure VM をセッション ホストとしてこのラボの前半でプロビジョニングしたホストプールに追加するために必要なファイルをホストするフォルダーを作成します。

   ```powershell
   $labFilesFolder = 'C:\AllFiles\Labs\02'
   New-Item -ItemType Directory -Path $labFilesFolder
   ```

>**注** PowerShell コマンドレットでコピーするときに [T] コンストラクトを使うことに注意してください。 場合によっては、$ 記号が 4 個の数字で示されるなど、コピーされるテキストが正しくないことがあります。 コマンドレットを発行する前に、これらを修正する必要があります。 PowerShell ISE の **[スクリプト]** ペインにコピーし、それらを修正してから、修正したテキストを強調表示にして **F8** キー ( **[選択範囲の実行]** ) を押します。

1. **az140-24-p3-0** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、セッション ホストをホスト プールに追加するために必要な Azure Virtual Desktop エージェントおよびブート ローダー インストーラーをダウンロードします。

   ```powershell
   $webClient = New-Object System.Net.WebClient
   $wvdAgentInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv'
   $wvdAgentInstallerName = 'WVD-Agent.msi'
   $webClient.DownloadFile($wvdAgentInstallerURL,"$labFilesFolder/$wvdAgentInstallerName")
   $wvdBootLoaderInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH'
   $wvdBootLoaderInstallerName = 'WVD-BootLoader.msi'
   $webClient.DownloadFile($wvdBootLoaderInstallerURL,"$labFilesFolder/$wvdBootLoaderInstallerName")
   ```

1. **az140-24-p3-0** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、PowerShellGet モジュールの最新バージョンをインストールします (確認を求められたら **[はい]** を選択します)。

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールから次のように実行して、最新バージョンの Az.DesktopVirtualization PowerShell モジュールをインストールします。

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -AllowClobber -Force
   Install-Module -Name Az -AllowClobber -Force
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、PowerShell 実行ポリシーを変更し、Azure サブスクリプションにサインインします。

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope CurrentUser -Force
   Connect-AzAccount
   ```

1. プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を入力します。
1. **az140-24-p3-0** へのリモート Desktopliveid セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから次を実行して、この演習の前半でプロビジョニングしたプールに新しいセッション ホストを参加させるために必要なトークンを生成します。

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName $resourceGroupName -HostPoolName $hostPoolName -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```
   > **注**:セッション ホストがプールに参加することを承認するには、登録トークンが必要です。 トークンの有効期限の値は、現在の日時から 1 時間から 1 か月の間である必要があります。

1. **az140-24-p3-0** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、Azure Virtual Desktop エージェントをインストールします。

   ```powershell
   Set-Location -Path $labFilesFolder
   Start-Process -FilePath 'msiexec.exe' -ArgumentList "/i $WVDAgentInstallerName", "/quiet", "/qn", "/norestart", "/passive", "REGISTRATIONTOKEN=$($registrationInfo.Token)", "/l* $labFilesFolder\AgentInstall.log" | Wait-Process
   ```

1. **az140-24-p3-0** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、Azure Virtual Desktop ブート ローダーをインストールします。

   ```powershell
   Start-Process -FilePath "msiexec.exe" -ArgumentList "/i $wvdBootLoaderInstallerName", "/quiet", "/qn", "/norestart", "/passive", "/l* $labFilesFolder\BootLoaderInstall.log" | Wait-process
   ```

#### <a name="task-5-verify-the-deployment-of-the-azure-virtual-desktop-host"></a>タスク 5:Azure Virtual Desktop ホストのデプロイを確認する

1. ラボ コンピューターに切り替え、Azure portal が表示されている Web ブラウザーで、**Azure Virtual Desktop** を検索して選択し、 **[Azure Virtual Desktop]** ブレードで **[ホスト プール]** を選択し、 **[Azure Virtual Desktop \| ホスト プール]** ブレードで、新しく変更されたプールを表すエントリ **az140-24-hp3** を選択します。
1. **[az140-24-hp3]** ブレードの左側にある垂直メニューの **[管理]** セクションで、 **[セッション ホスト]** をクリックします。 
1. **[az140-24-hp3 \| セッション ホスト]** ブレードで、デプロイに 1 つのホストが含まれていることを確認します。

#### <a name="task-6-manage-app-groups-using-powershell"></a>タスク 6:PowerShell を使用してアプリ グループを管理する

1. ラボ コンピューターから、リモート デスクトップ セッションを **az140-dc-vm11** に切り替え、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、リモート アプリ グループを作成します。

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $appGroupName = 'az140-24-hp3-Office365-RAG'
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   New-AzWvdApplicationGroup -Name $appGroupName -ResourceGroupName $resourceGroupName -ApplicationGroupType 'RemoteApp' -HostPoolArmPath "/subscriptions/$subscriptionId/resourcegroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName"-Location $location
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、プールのホスト上の **[スタート]** メニュー アプリを一覧表示し、出力を確認します。

   ```powershell
   Get-AzWvdStartMenuItem -ApplicationGroupName $appGroupName -ResourceGroupName $resourceGroupName | Format-List | more
   ```

   > **注**:公開するアプリケーションについては、**FilePath**、**IconPath**、**IconIndex** などのパラメーターを含む、出力に含まれる情報を記録する必要があります。

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから次のように実行して、Microsoft Word を発行します。

   ```powershell
   $name = 'Microsoft Word'
   $filePath = 'C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE'
   $iconPath = 'C:\Program Files\Microsoft Office\Root\VFS\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\wordicon.exe'
   New-AzWvdApplication -GroupName $appGroupName -Name $name -ResourceGroupName $resourceGroupName -FriendlyName $name -Filepath $filePath -IconPath $iconPath -IconIndex 0 -CommandLineSetting 'DoNotAllow' -ShowInPortal:$true
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから次のように実行して、Microsoft Word を発行します。

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-remote-app').Id
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName 'Desktop Virtualization User' -ResourceName $appGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

1. ラボ コンピューターに切り替え、Azure portal が表示されている Web ブラウザーの **[az140-24-hp3 \| セッション ホスト]** ブレードで、左側の垂直メニューの **[管理]** セクションの **[アプリケーション グループ]** を選びます。
1. **[az140-24-hp3 \| アプリケーション プール]** ブレードで、アプリケーション グループの一覧から **az140-24-hp3-Office365-RAG** エントリを選択します。
1. **[az140-24-hp3-Office365-RAG]** ブレードで、アプリケーションと割り当てを含むアプリケーション グループの構成を確認します。

### <a name="exercise-2-stop-and-deallocate-azure-vms-provisioned-in-the-lab"></a>演習 2:ラボでプロビジョニングされた Azure VM を停止および割り当て解除する

この演習の主なタスクは次のとおりです。

1. ラボでプロビジョニングされた Azure VM を停止および割り当て解除する

>**注**:この演習では、このラボでプロビジョニングした Azure VM を割り当て解除し、対応するコンピューティング料金を最小化します

#### <a name="task-1-deallocate-azure-vms-provisioned-in-the-lab"></a>タスク 1:ラボでプロビジョニングされた Azure VM を割り当て解除する

1. ラボ コンピューターに切り替え、Azure portal を表示している Web ブラウザーの画面で、**Cloud Shell** ペイン内の **PowerShell** シェル セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成されたすべての Azure VM を一覧表示します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG'
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成されたすべての Azure VM を停止して割り当てを解除します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注**:コマンドは非同期で実行されるので (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に停止されて割り当てが解除されるまでに数分かかります。
