---
lab:
  title: 'ラボ: セッション ホスト イメージ (AD DS) を作成して管理する'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# ラボ - セッション ホスト イメージ (AD DS) を作成して管理する
# 受講生用ラボ マニュアル

## ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプションの所有者または共同作成者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウントと、Azure サブスクリプションに関連付けられた Microsoft Entra テナントのグローバル管理者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウント。
- 完了したラボ: **Azure Virtual Desktop (AD DS) のデプロイを準備する**

## 推定所要時間

60 分

## ラボのシナリオ

Microsoft Entra DS 環境で Azure Virtual Desktop ホスト イメージを作成して管理する必要があります。

## 目標
  
このラボを完了すると、次のことができるようになります。

- WVD セッション ホスト イメージを作成して管理する

## ラボ ファイル

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json

## 手順

### 演習 1: セッション ホスト イメージを作成して管理する
  
この演習の主なタスクは次のとおりです。

1. Azure Virtual Desktop ホスト イメージの構成を準備する
1. Azure Bastion をデプロイする
1. Azure Virtual Desktop ホスト イメージを構成する
1. Azure Virtual Desktop ホスト イメージを作成する
1. カスタム イメージを使用して Azure Virtual Desktop ホスト プールをプロビジョニングする

#### タスク 1: Azure Virtual Desktop ホスト イメージの構成を準備する

1. ラボ コンピューターから Web ブラウザーを起動して [Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. Azure portal で、検索テキスト ボックスのすぐ右にあるツール バー アイコンを選択して **[Cloud Shell]** ペインを開きます。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、 **[PowerShell]** を選択します。 
1. ラボのコンピューターで、Azure portal を表示している Web ブラウザーで、[Cloud Shell] ペインの [PowerShell] セッションから次を実行して、Azure Virtual Desktop ホスト イメージを含むリソース グループを作成します。

   ```powershell
   $vnetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vnetResourceGroupName).Location
   $imageResourceGroupName = 'az140-25-RG'
   New-AzResourceGroup -Location $location -Name $imageResourceGroupName
   ```

1. Azure portal の Cloud Shell ペインのツール バーで、**[ファイルのアップロード/ダウンロード]** アイコンを選び、ドロップダウン メニューで **[アップロード]** を選んで、ファイル **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json** と **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json** を Cloud Shell のホーム ディレクトリにアップロードします。
1. [Cloud Shell] ペインの PowerShell セッションから、次を実行して、新しく作成されたサブネットに Azure Virtual Desktop クライアントとして機能する Windows 10 を実行する Azure VM をデプロイします。

   ```powershell
   New-AzResourceGroupDeployment `
     -ResourceGroupName $imageResourceGroupName `
     -Name az140lab0205vmDeployment `
     -TemplateFile $HOME/az140-25_azuredeployvm25.json `
     -TemplateParameterFile $HOME/az140-25_azuredeployvm25.parameters.json
   ```

   > **注**: このデプロイが完了するまで待ってから、次の演習に進んでください。 デプロイには約 10 分かかります。

#### タスク 2: Azure Bastion をデプロイする 

> **注**: Azure Bastion を使用すると、この演習の前のタスクでデプロイしたパブリック エンドポイントを使用せずに Azure VM に接続できると同時に、オペレーティング システム レベルの資格情報を対象とするブルート フォース攻撃から保護することができます。

> **注**: ブラウザーでポップアップ機能が有効になっていることを確認します。

1. Azure portal が表示されているブラウザー ウィンドウで、別のタブを開き、そのブラウザー タブで Azure portal に移動します。
1. Azure portal で、検索テキスト ボックスのすぐ右にあるツール バー アイコンを選択して **[Cloud Shell]** ペインを開きます。
1. Cloud Shell ペインの PowerShell セッションから以下を実行して、この演習で先ほど作成した **az140-25-vnet** という名前の仮想ネットワークに **AzureBastionSubnet** という名前のサブネットを追加します。

   ```powershell
   $resourceGroupName = 'az140-25-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-25-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.25.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. [Cloud Shell] ペインを閉じます。
1. Azure portal で **[複数の要塞]** を検索して選択し、**[複数の要塞]** ブレードから **[+ 追加]** を選択します。
1. **[Bastion の作成]** ウィンドウの **[基本]** タブで、次の設定を指定して、**[確認および作成]** を選びます。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |Resource group|**az140-25-RG**|
   |名前|**az140-25-bastion**|
   |リージョン|この演習の前のタスクでリソースをデプロイしたのと同じ Azure リージョン|
   |レベル|**Basic**|
   |仮想ネットワーク|**az140-25-vnet**|
   |サブネット|**AzureBastionSubnet (10.25.254.0/24)**|
   |パブリック IP アドレス|**新規作成**|
   |パブリック IP の名前|**az140-25-vnet-ip**|

1. **[Bastion の作成]** ウィンドウの **[確認と作成]** タブで、**[作成]** を選択します。

   > **注**: このデプロイが完了するまで待ってから、次の演習に進んでください。 デプロイには約 5 分かかります。

#### タスク 3: Azure Virtual Desktop ホスト イメージを構成する

1. Azure portal で、**[仮想マシン]** を見つけて選択し、**[仮想マシン]** ウィンドウで、**az140-25-vm0** を選択します。
1. **[az140-25-vm0]** ウィンドウで **[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、**[az140-25-vm0 \| 接続]** ウィンドウの **[Bastion]** タブで **[Bastion を使用する]** を選択します。
1. プロンプトが表示されたら、次の資格情報を入力し、**[接続]** を選択します。

   |設定|値|
   |---|---|
   |[ユーザー名]|**Student**|
   |パスワード|**Pa55w.rd1234**|

   > **注**: まず FSLogix バイナリをインストールします。

1. **az140-25-vm0** への Bastion セッション内で、**Windows PowerShell ISE** を管理者として起動します。
1. **az140-25-vm0** への Bastion セッション内で、[**Administrator: Windows PowerShell ISE**] コンソールから次を実行して、イメージを構成するための一時的な場所として使用するフォルダーを作成します。

   ```powershell
   New-Item -Type Directory -Path 'C:\Allfiles\Labs\02' -Force
   ```

1. **az140-25-vm0** への Bastion セッション内で、Microsoft Edge を起動し、[「FSLogix ダウンロード ページ」](https://aka.ms/fslogix_download)を参照し、FSLogix 圧縮インストール バイナリを **C:\\Allfiles\\Labs\\02** フォルダーにダウンロードし、ファイル エクスプローラーから **x64** サブフォルダーを同じフォルダーに抽出します。
1. **az140-25-vm0** への Bastion セッション内で、**[Administrator: Windows PowerShell ISE]** ウィンドウに切り替え、**[Administrator: Windows PowerShell ISE]** コンソールから次を実行して、OneDrive のマシンごとのインストールを実行します。

   ```powershell
   Start-Process -FilePath 'C:\Allfiles\Labs\02\x64\Release\FSLogixAppsSetup.exe' -ArgumentList '/quiet' -Wait
   ```

   > **注**: インストールが完了するまで待ちます。 これには 1 分ほどかかる場合があります。 インストールによって再起動がトリガーされた場合は、**az140-25-vm0** に再接続します。

   > **注**: 次に、Microsoft Teams のインストールと構成を段階的に行います (学習目的で、Teams はこのラボで使用されるイメージに既に存在するため)。

1. **az140-25-vm0** への Bastion セッション内で、**[スタート]** を右クリックし、右クリック メニューの **[実行]** を選択し、**[実行]** ダイアログ ボックスの **[開く]** テキストボックスに **「cmd」** と入力し、**Enter** キーを押して**コマンド プロンプト**を起動します。
1. **[Administrator: C:\windows\system32\cmd.exe]** ウィンドウで、コマンド プロンプトから次を実行して、Microsoft Teams のマシンごとのインストールの準備をします。

   ```cmd
   reg add "HKLM\Software\Microsoft\Teams" /v IsWVDEnvironment /t REG_DWORD /d 1 /f
   ```

1. **az140-25-vm0** への Bastion セッション内の Microsoft Edge で、[「Microsoft Visual C++ 再頒布可能パッケージのダウンロード ページ」](https://aka.ms/vs/16/release/vc_redist.x64.exe)を参照し、**VC_redist.x64** を **C:\\Allfiles\\Labs\\02** フォルダーに保存します。
1. **az140-25-vm0** への Bastion のセッション内で、**[Administrator: C:\windows\system32\cmd.exe]** ウィンドウに切り替え、コマンド プロンプトから次を実行して、[Microsoft Visual C++ 再頒布可能パッケージ] のインストールを実行します。

   ```cmd
   C:\Allfiles\Labs\02\vc_redist.x64.exe /install /passive /norestart /log C:\Allfiles\Labs\02\vc_redist.log
   ```

1. **az140-25-vm0** への Bastion セッション内の Microsoft Edge で[「Teams デスクトップ アプリを VM に展開する」](https://docs.microsoft.com/en-us/microsoftteams/teams-for-vdi#deploy-the-teams-desktop-app-to-the-vm)というタイトルのドキュメント ページを参照し、**64 ビット バージョン**のリンクをクリックして、プロンプトが表示されたら、**Teams_windows_x64.msi** ファイルを **C:\\Allfiles\\Labs\\02** フォルダーに保存します。
1. **az140-25-vm0** への Bastion のセッション内で、**[Administrator: C:\windows\system32\cmd.exe]** ウィンドウに切り替え、コマンド プロンプトから次を実行して、Microsoft Teams のマシンごとのインストールを実行します。

   ```cmd
   msiexec /i C:\Allfiles\Labs\02\Teams_windows_x64.msi /l*v C:\Allfiles\Labs\02\Teams.log ALLUSER=1
   ```

   > **注**: インストーラは ALLUSER=1 と ALLUSERS=1 パラメーターをサポートしています。 ALLUSER=1 パラメーターは、VDI 環境でのマシンごとのインストールを想定しています。 ALLUSERS=1 パラメーターは、VDI 以外の環境と VDI 環境で使用できます。 
   > **注**: **製品の別のバージョンが既にインストールされている**ことを示すエラーが発生した場合は、次の手順を実行します。**[コントロール パネル > プログラム > プログラムと機能]** から **[Teams コンピューター全体のインストーラー]** プログラムを右クリックし、**[アンインストール]** を選択します。 プログラムの削除を進めてから、上のステップ 13 を再実行します。 

1. **az140-25-vm0** への Bastion セッション内で、**[Windows PowerShell ISE]** を管理者として起動し、**[Administrator: Windows PowerShell ISE]** コンソールから次を実行して Microsoft Edge Chromium をインストールします (学習目的で、Edge はこのラボで使用されているイメージに既に存在するため)。

   ```powershell
   Start-BitsTransfer -Source "https://aka.ms/edge-msi" -Destination 'C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi'
   Start-Process -Wait -Filepath msiexec.exe -Argumentlist "/i C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi /q"
   ```

   > **注**: インストールが完了するまで待ちます。 これには 2 分ほどかかる場合があります。

   > **注**: 複数言語環境で動作する場合は、言語パックのインストールが必要になる場合があります。 この手順の詳細については、Microsoft Docs の記事「[Windows 10 マルチセッションイメージへの言語パックの追加](https://docs.microsoft.com/en-us/azure/virtual-desktop/language-packs)」を参照してください。

   > **注**: 次に、Windows 自動更新を無効にし、ストレージ センサーを無効にし、タイム ゾーン リダイレクトを構成し、テレメトリの収集を構成します。 一般に、すべての現在の更新プログラムを最初に適用する必要があります。 このラボでは、ラボの期間を最小限に抑えるために、この手順をスキップします。

1. **az140-25-vm0** への Bastion セッション内で、**[Administrator: C:\windows\system32\cmd.exe]** ウィンドウに切り替えて、コマンド プロンプトから次を実行して自動更新を無効にします。

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 1 /f
   ```

1. **[Administrator: C:\windows\system32\cmd.exe]** ウィンドウで、コマンド プロンプトから次を実行して、ストレージ センサーを無効にします。

   ```cmd
   reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\StorageSense\Parameters\StoragePolicy" /v 01 /t REG_DWORD /d 0 /f
   ```

1. **[Administrator: C:\windows\system32\cmd.exe]** ウィンドウで、コマンド プロンプトから次を実行して、タイム ゾーン リダイレクトを構成します。

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fEnableTimeZoneRedirection /t REG_DWORD /d 1 /f
   ```

1. **[Administrator: C:\windows\system32\cmd.exe]** ウィンドウで、コマンド プロンプトから次を実行して、テレメトリ データのフィードバック ハブ収集を無効にします。

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v AllowTelemetry /t REG_DWORD /d 0 /f
   ```

1. **[Administrator: C:\windows\system32\cmd.exe]** ウィンドウで、コマンド プロンプトから次を実行して、このタスクの前半で作成した一時フォルダーを削除します。

   ```cmd
   rmdir C:\Allfiles /s /q
   ```

1. **[Administrator: C:\windows\system32\cmd.exe]** ウィンドウで、コマンド プロンプトから、ディスク クリーンアップ ユーティリティを実行し、完了したら **[OK]** をクリックします。

   ```cmd
   cleanmgr /d C: /verylowdisk
   ```

#### タスク 4: Azure Virtual Desktop ホスト イメージを作成する

1. **az140-25-vm0** への Bastion セッション内の、**[Administrator: C:\windows\system32\cmd.exe]** ウィンドウで、コマンド プロンプトから、sysprep ユーティリティを実行して、オペレーティング・システムをイメージ生成用に準備し、自動的にシャットダウンします。

   ```cmd
   C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown /mode:vm
   ```

   > **注**: sysprep プロセスが完了するまでお待ちください。 これには 2 分ほどかかる場合があります。 これにより、オペレーティング システムが自動的にシャットダウンされます。 

1. ラボ コンピューターの Azure portal を表示する Web ブラウザーで、**[仮想マシン]** を見つけて選択し、**[仮想マシン]** ブレードから **az140-25-vm0** を選択します。
1. **[az140-25-vm0]** ブレードの **[要点]** セクションの上にあるツール バーで、**[更新]** をクリックし、Azure VM の**状態**が **[停止済み]** に変わったことを確認し、**[停止]** をクリックします。確認を求められたら、**[OK]** をクリックして Azure VM を**停止 (割り当て解除)** 状態に移行します。
1. **[az140-25-vm0]** ブレードで、Azure VM の**状態**が**停止 (割り当て解除)** 状態に変わったことを確認し、ツール バーの **[キャプチャ]** をクリックします。 **[イメージの作成]** ブレードが自動的に表示されます。
1. **[イメージの作成]** ブレードの **[基本]** タブで、次のように設定を行います。

   |設定|Value|
   |---|---|
   |Azure コンピューティング ギャラリーにイメージを共有する|**はい、イメージ バージョンとしてギャラリーに共有します**|
   |イメージの作成後、この仮想マシンを自動的に削除します|チェックボックスをオフにする|
   |ターゲット Azure コンピュート ギャラリー|新しいギャラリーの名前 **az14025imagegallery**|
   |オペレーティング システムの状態|**一般化されたイメージ**|

1. **[イメージの作成]** ブレードの **[基本]** タブで、**[ターゲット VM イメージ定義]** テキストボックスの下にある **[新規作成]** をクリックします。
1. **[VM イメージ定義の作成]** で、次の設定を指定して **[OK]** をクリックします。

   |設定|Value|
   |---|---|
   |VM イメージ定義名|**az140-25-host-image**|
   |発行元|**MicrosoftWindowsDesktop**|
   |プラン|**office-365**|
   |SKU|**win11-22h2-avd-m365**|

1. **[Bastion の作成]** ブレードの **[基本]** タブで、次の設定を指定して、**[確認および作成]** を選びます。

   |設定|Value|
   |---|---|
   |バージョン番号|**1.0.0**|
   |最新から除外|チェックボックスをオフにする|
   |有効期限の終了日|現在の日付から 1 年後|
   |既定のレプリカ数|**1**|
   |ターゲット リージョンのレプリカ数|**1**|
   |ストレージ アカウントの種類|**Premium SSD LRS**|

1. **[イメージの作成]** ブレードの **[確認と作成]** タブで、**[作成]** をクリックします。

   > **注**: デプロイが完了するまで待ちます。 これには 20 分ほどかかる場合があります。

1. ラボ コンピューターから、Azure portal を表示している Web ブラウザーで、**[Azure Compute Gallery]** を見つけて選択し、**[Azure Compute Gallery]** ウィンドウで **az14025imagegallery** エントリを選択し、****az14025imagegallery**** ウィンドウで新しく作成されたイメージを表す **az140-25-host-image** エントリの存在を確認します。

#### タスク 5: カスタム イメージを使用して Azure Virtual Desktop ホスト プールをプロビジョニングする

1. ラボのコンピューターの Azure portal で、Azure portal ページの上部にある **[リソース、サービス、およびドキュメントの検索]** テキスト ボックスを使用して、「**仮想ネットワーク**」と検索して移動し、**[仮想ネットワーク]** ブレードで **[az140-adds-vnet11]** を選択します。 
1. **[az140-adds-vnet11]** ウィンドウで **[サブネット]** を選択し、**[サブネット]** ウィンドウで **[+ サブネット]** を選択し、**[サブネットの追加]** ウィンドウで、次の設定を指定し (他のすべての設定は既定値のままにします)、**[保存]** をクリックします。

   |設定|値|
   |---|---|
   |名前|**hp4-Subnet**|
   |サブネットのアドレス範囲|**10.0.4.0/24**|

1. ラボ コンピューターから、Azure portal の Azure portal を表示する Web ブラウザーの画面で、**[Azure Virtual Desktop]** を見つけて選択し、**[Azure Virtual Desktop]** ウィンドウで **[ホスト プール]** を選択して、**[Azure Virtual Desktop \| ホスト プール]** ウィンドウで **[+ 作成]** を選びます。 
1. [**ホスト プールの作成**] ブレードの [**基本**] タブで、次の設定を指定して、[**次へ: 仮想マシン >**] を選択します。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |Resource group|**az140-25-RG**|
   |ホスト プール名|**az140-25-hp4**|
   |場所|このラボの最初の演習でリソースをデプロイした Azure リージョンの名前|
   |検証環境|**いいえ**|
   |ホスト プールの種類|**プールされた**|
   |最大セッションの制限|**50**|
   |負荷分散アルゴリズム|**幅優先**|

1. **[ホスト プールの作成]** ブレードの **[仮想マシン]** タブで、次の設定を指定します。

   |設定|Value|
   |---|---|
   |Azure 仮想マシンを追加する|**はい**|
   |Resource group|**既定はホスト プールと同じ**|
   |名前のプレフィックス|**az140-25-p4**|
   |仮想マシンの場所|このラボの最初の演習でリソースをデプロイした Azure リージョンの名前|
   |可用性オプション|**インフラストラクチャ冗長は必要ありません**|
   
1. **[ホスト プールの作成]** ブレードの **[仮想マシン]** タブの **[イメージ]** ドロップダウン リストのすぐ下にある **[すべてのイメージを表示]** リンクをクリックします。
1. **[イメージの選択]** ブレードの **[その他の項目]** で **[共有イメージ]** をクリックし、共有イメージの一覧で **[az140-25-host-image]** を選択します。 
1. **[ホスト プールの作成]** ブレードの **[仮想マシン]** タブに戻り、次の設定を指定し、**[Next: Workspace >]** を選択します。

   |設定|Value|
   |---|---|
   |[仮想マシンのサイズ]|**Standard D2s v3**|
   |[Number of VMs](VM の数)|**1**|
   |OS ディスクの種類|**Standard SSD**|
   |仮想ネットワーク|**az140-adds-vnet11**|
   |サブネット|**hp4-Subnet (10.0.4.0/24)**|
   |ネットワーク セキュリティ グループ|**Basic**|
   |パブリック インバウンド ポート|**はい**|
   |許可する受信ポート|**RDP**|
   |AD ドメイン参加 UPN|**student@adatum.com**|
   |Password|**Pa55w.rd1234**|
   |特定のドメインまたはユニット|**はい**|
   |参加するドメイン|**adatum.com**|
   |組織単位のパス|**OU=WVDInfra,DC=adatum,DC=com**|
   |ユーザー名|Student|
   |パスワード|Pa55w.rd1234|
   |[パスワードの確認入力]|Pa55w.rd1234|

1. **[ホスト プールの作成]** ブレードの **[ワークスペース]** タブで、次の設定を指定し、**[確認および作成]** を選択します。

   |設定|Value|
   |---|---|
   |デスクトップ アプリ グループを登録する|**いいえ**|

1. **[ホストプールの作成]** ブレードの **[確認および作成]** タブで、**[作成]** を選択します。

   > **注**: デプロイが完了するまで待ちます。 これには 10 分ほどかかる場合があります。
   > 
   > **注** クォータ制限に達したためにデプロイが失敗する場合は、最初のラボで説明した手順を実行して、Standard D2sv3 の制限を 30 に増やす要求を自動的に行います。

   > **注**: カスタム イメージに基づくホストのデプロイ後は、[GitHub リポジトリ](https://github.com/The-Virtual-Desktop-Team/)から入手できる Virtual Desktop 最適化ツールの実行を検討する必要があります。


### 演習 2: ラボでプロビジョニングされた Azure VM を停止して割り当てを解除する

この演習の主なタスクは次のとおりです。

1. ラボでプロビジョニングされた Azure VM を停止して割り当てを解除する

>**注**: この演習では、関連するコンピューティング料金を最小限に抑えるために、このラボでプロビジョニングされた Azure VM の割り当てを解除します

#### タスク 1: ラボでプロビジョニングされた Azure VM の割り当てを解除する

1. ラボ コンピューターに切り替え、Azure portal が表示されている Web ブラウザー ウィンドウで、[**Cloud Shell**] ウィンドウの **PowerShell** シェル セッションを開きます。
1. [Cloud Shell] ウィンドウの PowerShell セッションから、次を実行して、このラボで作成されたすべての Azure VM を一覧表示します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG'
   ```

1. [Cloud Shell] ウィンドウの PowerShell セッションから、次を実行して、このラボで作成したすべての Azure VM を停止して割り当てを解除します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注**: このコマンドは非同期で実行されるため (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に停止されて割り当てが解除されるまでに数分かかります。
