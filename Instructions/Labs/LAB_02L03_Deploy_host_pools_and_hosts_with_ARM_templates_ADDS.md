---
lab:
  title: 'ラボ:Azure Resource Manager テンプレート (AD DS) を使用してホスト プールおよびホストをデプロイする'
  module: 'Module 2: Implement a WVD Infrastructure'
---

# <a name="lab---deploy-host-pools-and-hosts-by-using-azure-resource-manager-templates"></a>ラボ- Azure Resource Manager テンプレートを使用してホスト プールと ホストをデプロイする
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-dependencies"></a>ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプション内で所有者または共同作成者のロールを持つ Microsoft アカウントまたは Azure AD アカウント、この Azure サブスクリプションに関連付けられた Azure AD テナント内でグローバル管理者ロールを持つMicrosoft アカウントまたは Azure AD アカウント。
- 実施するラボ - **Azure Virtual Desktop (AD DS) のデプロイを準備する**
- 実施するラボ - **Azure portal (AD DS) を使用してホスト プールとセッション ホストをデプロイする**

## <a name="estimated-time"></a>推定所要時間

45 分

## <a name="lab-scenario"></a>ラボのシナリオ

Azure Resource Manager テンプレートを使用して、Azure Virtual Desktop ホスト プールとホストのデプロイを自動化する必要があります。

## <a name="objectives"></a>目標
  
このラボを完了すると、次のことができるようになります。

- Azure Resource Manager テンプレートを使用して、Azure Virtual Desktop ホスト プールとホストをデプロイする

## <a name="lab-files"></a>ラボ ファイル 

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json

## <a name="instructions"></a>手順

### <a name="exercise-1-deploy-azure-virtual-desktop-host-pools-and-hosts-by-using-azure-resource-manager-templates"></a>演習 1:Azure Resource Manager テンプレートを使用して、Azure Virtual Desktop ホスト プールとホストをデプロイする
  
この演習の主なタスクは次のとおりです。

1. Azure Resource Manager テンプレートを使用して、Azure Virtual Desktop ホスト プールのデプロイを準備する
1. Azure Resource Manager テンプレートを使用して、Azure Virtual Desktop ホスト プールと ホストをデプロイする
1. Azure Virtual Desktop ホスト プールとホストのデプロイを確認する
1. Azure Resource Manager テンプレートを使用して、既存の Azure Virtual Desktop ホスト プールに追加するホストを準備する
1. Azure Resource Manager テンプレートを使用して、既存の Azure Virtual Desktop ホスト プールにホストを追加する
1. Azure Virtual Desktop ホスト プールに対する変更を確認する
1. Azure Virtual Desktop ホスト プールで個人用デスクトップの割り当てを管理する

#### <a name="task-1-prepare-for-deployment-of-an-azure-virtual-desktop-host-pool-by-using-an-azure-resource-manager-template"></a>タスク 1:Azure Resource Manager テンプレートを使用して、Azure Virtual Desktop ホスト プールのデプロイを準備する

1. ラボのコンピューターから Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者の役割を持つユーザーアカウントの認証情報を提供してサインインします。
1. Azure portal で、「**仮想マシン**」を検索して選択し、 **[Virtual Machines]** ブレードで、 **[az140-vm11]** を選択します。
1. **[az140-dc-vm11]** ウィンドウで **[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、 **[az140-dc-vm11 \| 接続]** ウィンドウの **[Bastion]** タブで **[Bastion を使用する]** を選択します。
1. プロンプトが表示されたら、次の資格情報を入力し、 **[接続]** を選択します。

   |設定|値|
   |---|---|
   |[ユーザー名]|**学生**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、**Windows PowerShell ISE** を管理者として起動します。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、Azure Virtual Desktop プール ホストのコンピューター オブジェクトをホストする **WVDInfra** という名前の組織単位の識別名を特定します。

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、Azure Virtual Desktop ホストを AD DS ドメイン ( **student@adatum.com** ) に参加させるために使用する **ADATUM\\Student** アカウントのユーザー プリンシパル名属性を特定します。

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'student'").userPrincipalName
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、このラボの後半で個人用デスクトップの割り当てをテストするために使用する **ADATUM\\aduser7** および **ADATUM\\aduser8** アカウントのユーザー プリンシパル名を特定します。

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser7'").userPrincipalName
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser8'").userPrincipalName
   ```

   > **注**:識別したすべてのユーザー プリンシパル名の値を記録します。 これらは、このラボの後半で必要になります。

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、テンプレートベースのデプロイを実行するために必要なトークンの有効期限を計算します。

   ```powershell
   $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

   > **注**:値は、`2022-03-27T00:51:28.3008055Z` のような形式になります。 次のタスクで必要になるので、記録します。

   > **注**:ホストがプールに参加することを承認するには、登録トークンが必要です。 トークンの有効期限の値は、現在の日時から 1 時間から 1 か月の間である必要があります。

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Microsoft Edge を起動して、[Azure portal](https://portal.azure.com) に移動します。 プロンプトが表示されたら、このラボで使用しているサブスクリプションで所有者の役割を持つユーザーアカウントの資格情報を使用してサインインします。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal の Azure portal ページの上部にある **[リソース、サービス、およびドキュメントの検索]** テキスト ボックスを使用して、「**仮想ネットワーク**」と検索して移動し、 **[仮想ネットワーク]** ブレードで **[az140-adds-vnet11]** を選択します。 
1. **[az140-adds-vnet11]** ブレードで **[サブネット]** を選択し、 **[サブネット]** ブレードで **[+ サブネット]** を選択し、 **[サブネットの追加]** ブレードで次の設定を指定し (他のすべての設定は既定値のままにします)、 **[保存]** をクリックします。

   |設定|値|
   |---|---|
   |名前|**hp2-Subnet**|
   |サブネットのアドレス範囲|**10.0.2.0/24**|

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal で、Azure portal ページの上部にある **[リソース、サービス、およびドキュメントの検索]** テキストボックスを使用して、**ネットワーク セキュリティ グループ**を検索して移動し、 **[ネットワーク セキュリティ グループ]** ブレードで、**az140-11-RG** リソース グループでネットワーク セキュリティ グループを選択します。
1. ネットワーク セキュリティ グループ ブレードの左側の垂直メニューの **[設定]** セクションで、 **[プロパティ]** をクリックします。
1. **[プロパティ]** ブレードで、 **[リソース ID]** テキストボックスの右側にある **[クリップボードにコピー]** アイコンをクリックします。 

   > **注**:値は、`/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg` のような形式になります。ただし、サブスクリプション ID は異なります。 次のタスクで必要になるので、記録します。

#### <a name="task-2-deploy-an-azure-virtual-desktop-host-pool-and-hosts-by-using-an-azure-resource-manager-template"></a>タスク 2:Azure Resource Manager テンプレートを使用して、Azure Virtual Desktop ホスト プールと ホストをデプロイする

1. ラボのコンピューターから Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者の役割を持つユーザーアカウントの認証情報を提供してサインインします。
1. ラボ コンピューターから、同じ Web ブラウザー ウィンドウで、別の Web ブラウザー タブを開き、GitHub Azure RDS テンプレート リポジトリ ページの [ARM テンプレートに移動して、新しい Azure Virtual Desktop ホスト プールを作成およびプロビジョニングします](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/CreateAndProvisionHostPool)。 
1. **[新しい Azure Virtual Desktop ホスト プールを作成してプロビジョニングするための ARM テンプレート]** ページで、 **[Azure にデプロイ]** を選択します。 これにより、ブラウザーが Azure portal の **[カスタム デプロイ]** ブレードに自動的にリダイレクトされます。
1. **[カスタム デプロイ]** ブレードで、 **[パラメーターの編集]** を選択します。
1. **[パラメーターの編集]** ウィンドウで、 **[ファイルの読み込み]** を選択し、 **[開く]** ダイアログ ボックスで、 **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json** を選択し、 **[開く]** を選択して、 **[保存]** を選択します。 
1. 再び **[カスタム デプロイ]** ブレードで以下の設定を指定します (他は既存の値のままにします):

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|新しいリソース グループの名前 **az140-23-RG**|
   |Region|ラボ「**Azure Virtual Desktop (AD DS) のデプロイを準備する**」で AD DS ドメイン コントローラーをホストする Azure VM をデプロイした Azure リージョンの名前|
   |場所|**Region** パラメーターの値として設定されたものと同じ Azure リージョンの名前|
   |ワークスペースの場所|**Region** パラメーターの値として設定されたものと同じ Azure リージョンの名前|
   |ワークスペース リソース グループ|null の場合、その値はデプロイ ターゲット リソース グループに一致するように自動的に設定されるため、なし|
   |すべてのアプリケーション グループ リファレンス|ターゲット ワークスペースに既存のアプリケーション グループがないため、なし (ワークスペースがない)|
   |仮想マシンの場所|**Location** パラメーターの値として設定されたものと同じ Azure リージョンの名前|
   |ネットワーク セキュリティ グループの作成|**false**|
   |ネットワーク セキュリティ グループ ID|前のタスクで特定した既存のネットワーク セキュリティ グループの resourceID パラメーターの値|
   |トークンの有効期限| 前のタスクで計算したトークンの有効期限の値|

   > **注**:デプロイでは、個人用デスクトップ割り当てタイプのプールがプロビジョニングされます。

1. **[カスタム デプロイ]** ブレードで、 **[Review + create]** をクリックし、 **[作成]** をクリックします。

   > **注**: デプロイが完了するまで待ってから、次のタスクに進んでください。 これには 15 分ほどかかる場合があります。 

#### <a name="task-3-verify-deployment-of-the-azure-virtual-desktop-host-pool-and-hosts"></a>タスク 3:Azure Virtual Desktop ホスト プールとホストのデプロイを確認する

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザーで、「**Azure Virtual Desktop**」を検索して選択し、 **[Azure Virtual Desktop]** ウィンドウで **[ホスト プール]** を選択し、 **[Azure Virtual Desktop \| ホスト プール]** ウィンドウで、新しく変更されたプールを表すエントリ **[az140-23-hp2]** を選択します。
1. **[az140-23-hp2]** ブレードの左側にある垂直メニューの **[管理]** セクションで、 **[セッション ホスト]** をクリックします。 
1. **[az140-23-hp2 \| セッション ホスト]** ウィンドウで、デプロイが 2 つのホストで構成されていることを確認します。
1. **[az140-23-hp2 \| セッション ホスト]** ウィンドウの左側の垂直メニューにある **[管理]** セクションで、 **[アプリケーション グループ]** をクリックします。
1. **[az140-23-hp2 \| アプリケーション グループ]** ウィンドウで、**az140-23-hp2-DAG** という名前の "**既定のデスクトップ**" アプリケーション グループがデプロイに含まれていることを確認します。

#### <a name="task-4-prepare-for-adding-of-hosts-to-the-existing-azure-virtual-desktop-host-pool-by-using-an-azure-resource-manager-template"></a>タスク 4:Azure Resource Manager テンプレートを使用して、既存の Azure Virtual Desktop ホスト プールに追加するホストを準備する

1. ラボ コンピューターから、リモート デスクトップ セッションを **az140-dc-vm11** に切り替えます。 
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、この演習で先ほどプロビジョニングしたプールに新しいホストを参加させるために必要なトークンを生成します。

   ```powershell
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName 'az140-23-RG' -HostPoolName 'az140-23-hp2' -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行してトークンの値を取得し、クリップボードに貼り付けます。

   ```powershell
   $registrationInfo.Token | clip
   ```

   > **注**:クリップボードにコピーされた値を記録します (たとえば、メモ帳を起動し、Ctrl+V キーの組み合わせを押して、クリップボードの内容をメモ帳に貼り付けます)。次のタスクで必要になるためです。 使用している値に、改行なしで 1 行のテキストが含まれていることを確認してください。 

   > **注**:ホストがプールに参加することを承認するには、登録トークンが必要です。 トークンの有効期限の値は、現在の日時から 1 時間から 1 か月の間である必要があります。

#### <a name="task-5-add-hosts-to-the-existing-azure-virtual-desktop-host-pool-by-using-an-azure-resource-manager-template"></a>タスク 5:Azure Resource Manager テンプレートを使用して、既存の Azure Virtual Desktop ホスト プールにホストを追加する

1. ラボ コンピューターから、同じ Web ブラウザー ウィンドウで、別の Web ブラウザー タブを開き、GitHub Azure RDS テンプレート リポジトリ ページの [ARM テンプレートに移動して、セッションホストを既存の Azure Virtual Desktop ホストプールに追加します](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/AddVirtualMachinesToHostPool)。 
1. **[セッションホストを既存の Azure Virtual Desktop ホストプールに追加するための ARM テンプレート]** ページで、 **[Azure にデプロイ]** を選択します。 これにより、ブラウザーが Azure portal の **[カスタム デプロイ]** ブレードに自動的にリダイレクトされます。
1. **[カスタム デプロイ]** ブレードで、 **[パラメーターの編集]** を選択します。
1. **[パラメーターの編集]** ウィンドウで、 **[ファイルの読み込み]** を選択し、 **[開く]** ダイアログ ボックスで、 **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json** を選択し、 **[開く]** を選択して、 **[保存]** を選択します。 
1. 再び **[カスタム デプロイ]** ブレードで以下の設定を指定します (他は既存の値のままにします):

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**az140-23-RG**|
   |ホストプール トークン|前のタスクで生成したトークンの値|
   |ホストプールの場所|このラボの前半でホストプールをデプロイした Azure リージョンの名前|
   |VM 管理者アカウントのユーザー名|**student** @adatum.com を使用しないでください|
   |VM 管理者アカウントのパスワード|**Pa55w.rd1234**|
   |仮想マシンの場所|**Hostpool Location** パラメーターの値として設定されたものと同じ Azure リージョンの名前|
   |ネットワーク セキュリティ グループの作成|**false**|
   |ネットワーク セキュリティ グループ ID|前のタスクで特定した既存のネットワーク セキュリティ グループの resourceID パラメーターの値|

1. **[カスタム デプロイ]** ブレードで、 **[Review + create]** をクリックし、 **[作成]** をクリックします。

   > **注**: デプロイが完了するまで待ってから、次のタスクに進んでください。 これには 5 分ほどかかる場合があります。

#### <a name="task-6-verify-changes-to-the-azure-virtual-desktop-host-pool"></a>タスク 6:Azure Virtual Desktop ホスト プールに対する変更を確認する

1. ラボ コンピューターの Azure portal を表示している Web ブラウザーで、「**仮想マシン**」を検索して選択し、 **[仮想マシン]** ウィンドウで、一覧に **az140-23-p2-2** という名前の追加の仮想マシンが含まれていることに注意してください。
1. ラボ コンピューターから、リモート デスクトップ セッションを **az140-dc-vm11** に切り替えます。 
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、3 番目のホストが **adatum.com** AD DS ドメインに正常に参加したことを確認します。

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-23-p2-2$'"
   ```
1. ラボ コンピューターに戻り、Azure portal が表示されている Web ブラウザーで、「**Azure Virtual Desktop**」を検索して選択し、 **[Azure Virtual Desktop]** ウィンドウで **[ホスト プール]** を選択し、 **[Azure Virtual Desktop \| ホスト プール]** ウィンドウで、新しく変更されたプールを表すエントリ **[az140-23-hp2]** を選択します。
1. **[az140-23-hp2]** ブレードで、 **[Essentials]** セクションを確認し、 **[ホスト プールの種類]** が **[個人用]** に設定され、 **[割り当ての種類]** が **[自動]** に設定されていることを確認します。
1. **[az140-23-hp2]** ブレードの左側にある垂直メニューの **[管理]** セクションで、 **[セッション ホスト]** をクリックします。 
1. **[az140-23-hp2 \| セッション ホスト]** ウィンドウで、デプロイが 3 つのホストで構成されていることを確認します。 

#### <a name="task-7-manage-personal-desktop-assignments-in-the-azure-virtual-desktop-host-pool"></a>タスク 7:Azure Virtual Desktop ホスト プールで個人用デスクトップの割り当てを管理する

1. ラボ コンピューターの Azure portal が表示されている Web ブラウザーの **[az140-23-hp2 \| セッション ホスト]** ウィンドウの左側の垂直メニューにある **[管理]** セクションで、 **[アプリケーション グループ]** を選択します。 
1. **[az140-23-hp2 \| アプリケーション グループ]** ウィンドウのアプリケーション グループの一覧で **[az140-23-hp2-DAG]** を選択します。
1. **[az140-23-hp2-DAG]** ブレードの左側の垂直メニューで、 **[割り当て]** を選択します。 
1. **[az140-23-hp2-DAG \| 割り当て]** ウィンドウで、 **[+ 追加]** を選択します。
1. **[Azure AD ユーザーまたはユーザー グループの選択]** ブレードで、 **[az140-wvd-personal]** を選択し、 **[選択]** をクリックします。

   > **注**:次に、Azure Virtual Desktop ホスト プールに接続しているユーザーのエクスペリエンスを確認しましょう。

1. ラボ コンピューターから、Azure portal を表示している Web ブラウザーの画面で、**仮想ネットワーク**を検索して選択し、 **[仮想ネットワーク]** ブレードから **az140-cl-vm11** エントリを選択します。
1. **[az140-cl-vm11]** ウィンドウで **[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、 **[az140-cl-vm11 \| 接続]** ウィンドウの **[Bastion]** タブで **[Bastion を使用する]** を選択します。
1. プロンプトが表示されたら、次の資格情報を入力し、 **[接続]** を選択します。

   |設定|値|
   |---|---|
   |[ユーザー名]|**Student@adatum.com**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-cl-vm11** へのリモート デスクトップ セッション内で、 **[スタート]** をクリックし、 **[スタート]** メニューで、**リモート デスクトップ** クライアント アプリを選択します。
2. [リモート デスクトップ] ウィンドウで、右上隅にある省略記号アイコンをクリックし、ドロップダウン メニューで **[登録を解除]** をクリックし、確認のメッセージが表示されたら、 **[続行]** をクリックします。
3. **az140-cl-vm11** へのリモート デスクトップ セッション内で、[リモート デスクトップ] ウィンドウの **[始めましょう]** ページで、 **[登録する]** をクリックします。
4. **リモート デスクトップ** クライアント ウィンドウで、 **[サブスクライブ]** を選択し、プロンプトが表示されたら、**aduser7** 資格情報を使用してサインインします (userPrincipalName と、パスワードとして **Pa55w.rd1234** を指定します)。

   > **注**:または、 **[リモート デスクトップ]** クライアント ウィンドウで **[URL で登録]** を選択し、 **[ワークスペースへの登録]** ペインの **[メールまたはワークスペース URL]** に「 **https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery** 」と入力して、 **[次へ]** を選択します。プロンプトが表示されたら、**aduser7** の資格情報を使用してサインインします (userPrincipalName 属性をユーザー名として使用し、このアカウントの作成時に設定したパスワードを使用します)。 

1. **[リモート デスクトップ]** ページで、**SessionDesktop** アイコンをダブルクリックし、資格情報の入力を求められたら、同じパスワードをもう一度入力し、 **[記憶する]** チェックボックスをオンにして、 **[OK]** をクリックします。
1. **[すべてのアプリにサインインしたままにする]** ウィンドウで、 **[組織がデバイスを管理できるようにする]** チェックボックスをオフにし、 **[いいえ、このアプリのみにサインインします]** を選択します。 
1. **aduser7** がリモート デスクトップ経由でホストに正常にサインインしたことを確認します。
1. **aduser7** としてホストの 1 つへのリモート デスクトップ セッション内で、 **[スタート]** を右クリックし、右クリック メニューで **[シャットダウン] または [サインアウト]** を選択し、カスケード メニューで **[サインアウト]** をクリックします。

   > **注**:次に、個人用デスクトップの割り当てを直接モードから自動モードに切り替えましょう。 

1. ラボ コンピューター、Azure portal を表示する Web ブラウザーの順に切り替えて、 **[az140-23-hp2-DAG \| 割り当て]** ウィンドウの割り当ての一覧のすぐ上にある情報バーで **[VM の割り当て]** リンクをクリックします。 これにより、 **[az140-23-hp2 \| セッション ホスト]** ウィンドウにリダイレクトされます。 
1. **[az140-23-hp2 \| セッション ホスト]** で、ホストの 1 つの **[割り当て済みユーザー]** 列に **aduser7** が一覧表示されていることを確認します。

   > **注**:ホスト プールは自動割り当て用に構成されているため、これは予想されます。

1. ラボ コンピューターの Azure portal を表示している Web ブラウザーの画面で、**Cloud Shell** ペイン内の **PowerShell** シェル セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して直接割り当てモードに切り替えます。

    ```powershell
    Update-AzWvdHostPool -ResourceGroupName 'az140-23-RG' -Name 'az140-23-hp2' -PersonalDesktopAssignmentType Direct
    ```

1. ラボ コンピューターで、Azure portal を表示している Web ブラウザーの画面で、**az140-23-hp2** ホスト プール ブレードに移動し、 **[Essentials]** セクションを確認して、 **[ホスト プールの種類]** が **[個人用]** に設定され、 **[割り当ての種類]** が **[直接]** に設定されていることを確認します。
1. リモート デスクトップ セッションに戻って **az140-cl-vm11** に切り替え、 **[リモート デスクトップ]** ウィンドウの右上隅にある省略記号アイコンをクリックし、ドロップダウン メニューで **[登録を解除]** をクリックし、確認を求めるメッセージが表示されたら、 **[続行]** をクリックします。
1. **az140-cl-vm11** へのリモート デスクトップ セッション内の **[リモート デスクトップ]** ウィンドウの **[はじめに]** ページで、 **[購読]** をクリックします。
1. サインインするように求められたら、 **[アカウントの選択]** ペインで、 **[別のアカウントを使用する]** をクリックし、プロンプトが表示されたら、**aduser8** ユーザー アカウントのユーザー プリンシパル名と、このアカウントの作成時に設定したパスワードを使用してサインインします。
1. **[すべてのアプリにサインインしたままにする]** ウィンドウで、 **[組織がデバイスを管理できるようにする]** チェックボックスをオフにし、 **[いいえ、このアプリのみにサインインします]** を選択します。 
1. **[リモート デスクトップ]** ページで、 **[SessionDesktop]** アイコンをダブルクリックし、**現在利用可能なリソースがないために接続できなかったことを示すエラー メッセージが表示されることを確認します。後で再試行するか、これが引き続き発生する場合はテクニカルサポートに連絡して**、 **[OK]** をクリックしてください。

   > **注**:これは、ホスト プールが直接割り当て用に構成されており、**aduser8** にホストが割り当てられていないためです。

1. ラボ コンピューター、Azure portal を表示する Web ブラウザーの順に切り替えて、 **[az140-23-hp2 \| セッション ホスト]** で、割り当てられていない残り 2 つのホストのいずれかの横にある **[割り当て済みユーザー]** 列の **(割り当て)** リンクを選択します。
1. **[ユーザーの割り当て]** で、 **[aduser8]** を選択し、 **[選択]** をクリックして、確認を求められたら、 **[OK]** をクリックします。
1. **az140-cl-vm11**へのリモート デスクトップ セッションに戻って、 **[リモート デスクトップ]** ウィンドウで、 **[SessionDesktop]** アイコンをダブルクリックし、パスワードの入力を求められたら、このユーザー アカウントの作成時に設定したパスワードを入力し、 **[OK]** をクリックして、割り当てられたホストに正常にサインインできることを確認します。

### <a name="exercise-2-stop-and-deallocate-azure-vms-provisioned-in-the-lab"></a>演習 2:ラボでプロビジョニングされた Azure VM を停止および割り当て解除する

この演習の主なタスクは次のとおりです。

1. ラボでプロビジョニングされた Azure VM を停止および割り当て解除する

>**注**:この演習では、このラボでプロビジョニングした Azure VM を割り当て解除し、対応するコンピューティング料金を最小化します

#### <a name="task-1-deallocate-azure-vms-provisioned-in-the-lab"></a>タスク 1:ラボでプロビジョニングされた Azure VM を割り当て解除する

1. ラボ コンピューターに切り替え、Azure portal を表示している Web ブラウザーの画面で、**Cloud Shell** ペイン内の **PowerShell** シェル セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成されたすべての Azure VM を一覧表示します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG'
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成されたすべての Azure VM を停止して割り当てを解除します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注**:コマンドは非同期で実行されるので (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に停止されて割り当てが解除されるまでに数分かかります。
