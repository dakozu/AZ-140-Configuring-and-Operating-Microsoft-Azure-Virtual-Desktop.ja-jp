---
lab:
  title: 'ラボ: Azure Virtual Desktop プロファイル (AD DS) を実装して管理する'
  module: 'Module 4: Manage User Environments and Apps'
---

# ラボ - Azure Virtual Desktop プロファイル (AD DS) を実装して管理する
# 受講生用ラボ マニュアル

## ラボの依存関係

- このラボで使用する Azure サブスクリプション。
- このラボで使用する Azure サブスクリプションの所有者または共同作成者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウントと、Azure サブスクリプションに関連付けられた Microsoft Entra テナントのグローバル管理者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウント。
- 完了したラボ: **Azure Virtual Desktop (AD DS) のデプロイを準備する**
- 完成したラボ「**AVD (AD DS) 用のストレージを実装および管理する**」

## 推定所要時間

30 分

## ラボのシナリオ

Active Directory Domain Services (AD DS) 環境で Azure Virtual Desktop プロファイル管理を実装する必要があります。

## 目標
  
このラボを完了すると、次のことができるようになります。

- Azure Virtual Desktop 用に FSLogix ベースのプロファイルを実装する

## ラボ ファイル

- なし

## 手順

### 演習 1: Azure Virtual Desktop 用に FSLogix ベースのプロファイルを実装する

この演習の主なタスクは次のとおりです。

1. Azure Virtual Desktop セッション ホスト VM で FSLogix ベースのプロファイルを構成する
1. Azure Virtual Desktop 用に FSLogix ベースのプロファイルをテストする

#### タスク 1: Azure Virtual Desktop セッション ホスト VM で FSLogix ベースのプロファイルを構成する

1. ラボ コンピューターから Web ブラウザーを起動して [Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. Azure portal で、検索テキスト ボックスのすぐ右にあるツール バー アイコンを選択して [**Cloud Shell**] ウィンドウを開きます。
1. **[Cloud Shell]** ペインで次を実行して、このラボで使用する Azure VM をホストする Azure Virtual Desktop セッションを開始します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**注**: 次の手順に進む前に、Azure VM が実行されるまで待ちます。

1. **[Cloud Shell]** ペインで次を実行して、セッション ホストでの PowerShell リモート処理を有効にします。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```

1. Cloud Shell を閉じる
1. Azure portal で、「**仮想マシン**」を検索して選択し、**[仮想マシン]** ブレードで **[az140-21-p1-0]** を選択します。
1. **az140-21-p1-0** ブレードで **[接続]** を選択し、ドロップダウン メニューで **[Bastion 経由で接続する]** を選択します。
1. プロンプトが表示されたら、次の認証情報を入力します。

   |設定|値|
   |---|---|
   |ユーザー名|**student@adatum.com**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-21-p1-0** への Bastion セッション内で、Microsoft Edge を起動し、[「FSLogix ダウンロード ページ」](https://aka.ms/fslogix_download)を参照し、FSLogix 圧縮インストール バイナリをダウンロードして、これを **C:\\Allfiles\\Labs\\04** フォルダー (必要に応じて、フォルダーを作成する) に抽出し、**x64\\Release** サブフォルダーに移動して、**FSLogixAppsSetup.exe** ファイルをダブルクリックし、**Microsoft FSLogix Apps セットアップ**ウィザードを起動して、既定の設定で Microsoft FSLogix Apps をインストールします。

   > **注**:イメージがすでに含まれている場合、FSLogix のインストールは不要です。

1. **az140-21-p1-0** への Bastion セッション内で、**Windows PowerShell ISE** を管理者として起動します。
1. **[管理者: Windows PowerShell ISE]** コンソールから次を実行して、最新バージョンの Az PowerShell モジュールをインストールします (NuGet のインストールとインポートを求められたら、「**Y**」と入力します)。

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck -Force
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

1. プロンプトが表示されたら、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの Microsoft Entra 資格情報を指定してサインインします。
1. **az140-21-p1-0** への Bastion セッション内において、**[Administrator: Windows PowerShell ISE]** スクリプト ペインで、次を実行して、このラボで以前に構成した Azure ストレージ アカウントの名前を取得します。

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName
   ```

1. **az140-21-p1-0** への Bastion セッション内の **[Administrator: Windows PowerShell ISE]** スクリプト ペインで、次を実行して、プロファイル レジストリ設定を構成します。

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```

1. **az140-21-p1-0** への Bastion セッション内で、**[スタート]** を右クリックし、右クリック メニューの **[実行]** を選択し、**[実行]** ダイアログ ボックスの **[開く]** テキスト ボックスに次のように入力し、**[OK]** を選択して **[ローカル ユーザーとグループ]** コンソールを起動します。

   ```cmd
   lusrmgr.msc
   ```

1. **ローカル ユーザーとグループ** コンソールで、**FSLogix** 文字列で始まる名前の 4 つのグループに注意してください。

   - FSLogix ODFC Exclude List
   - FSLogix ODFC Include List
   - FSLogix Profile Exclude List
   - FSLogix Profile Include List

1. **[ローカル ユーザーとグループ]** コンソールのグループのリストで、**[FSLogix Profile Include Lis]** グループをダブルクリックし、**\\Everyone** グループが含まれていることを確認し、**[OK]** を選択して、グループの **[プロパティ]** ウィンドウを閉じます。 
1. **[ローカル ユーザーとグループ]** コンソールのグループのリストで、**[FSLogix Profile Exclude List]** グループをダブルクリックし (既定では、グループ メンバーは含まれていません)、**[OK]** を選択して、グループの **[プロパティ]** ウィンドウを閉じます。 

   > **注**: 一貫性のあるユーザー エクスペリエンスを提供するには、すべての Azure Virtual Desktop セッション ホストに FSLogix コンポーネントをインストールして構成する必要があります。 このタスクは、ラボ環境の他のセッション ホストで無人で実行します。 

   > **注**:FSLogix がセッション ホストに既にインストールされている場合、次の手順は必要ありません。

1. **az140-21-p1-0** への Bastion セッション内で、**[Administrator: Windows PowerShell ISE]** スクリプト ペインで、次を実行して、FSLogix コンポーネントを **az140-21-p1-1** と **az140-21-p1-2** セッション ホストにインストールします。

   ```powershell
   $servers = 'az140-21-p1-1', 'az140-21-p1-2'
   foreach ($server in $servers) {
      $localPath = 'C:\Allfiles\Labs\04\x64'
      $remotePath = "\\$server\C$\Allfiles\Labs\04\x64\Release"
      Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
      Invoke-Command -ComputerName $server -ScriptBlock {
         Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
      } 
   }
   ```

   > **注**: スクリプトの実行が完了するまで待ちます。 これには 2 分ほどかかる場合があります。

1. **az140-21-p1-0** への Bastion セッション内で、**[Administrator: Windows PowerShell ISE]** スクリプト ペインで、次を実行して、**az140-21-p1-1** と **az140-21-p1-1** セッション ホストのプロファイル レジストリ設定を構成します。

   ```powershell
   $servers = 'az140-21-p1-1', 'az140-21-p1-2'
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   foreach ($server in $servers) {
      Invoke-Command -ComputerName $server -ScriptBlock {
         New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$using:storageAccountName.file.core.windows.net\$using:fileShareName"
      }
   }
   ```

   > **注**: FSLogix ベースのプロファイル機能をテストする前に、テストに使用する **ADATUM\\aduser1** アカウントのローカルにキャッシュされたプロファイルを、前のラボで使用した Azure Virtual Desktop セッション ホストから削除する必要があります。

1. **az140-21-p1-0** への Bastion セッション内で、**[Administrator: Windows PowerShell ISE]** スクリプト ペインで、次を実行して、セッションホストとして機能するすべての Azure VM でローカルにキャッシュされた **ADATUM\\aduser1** アカウントのプロファイルを削除します。

   ```powershell
   $userName = 'aduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1', 'az140-21-p1-2'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

1. **az140-21-p1-0** への Bastion セッション内で **[スタート]** を右クリックし、右クリック メニューで **[シャットダウンまたはサインアウト]** を選択して、カスケード メニューで **[サインアウト]** を選択します。
1. **[切断]** ウィンドウから、**[閉じる]** を選択します。

#### タスク 2: Azure Virtual Desktop を使用して FSLogix ベースのプロファイルをテストする

1. ラボ コンピューターから、Azure portal を表示している Web ブラウザーの画面を表示しているラボ コンピューターに切り替え、「**仮想マシン**」を検索して選択し、**[仮想ネットワーク]** ブレードから **[az140-cl-vm11]** エントリを選択します。
1. **[az140-cl-vm11]** ブレードで **[接続]** を選択し、ドロップダウン メニューで **[Bastion 経由で接続する]** を選択します。
1. プロンプトが表示されたら、次の資格情報を入力し、**[接続]** を選択します。

   |設定|値|
   |---|---|
   |ユーザー名|**Student@adatum.com**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-cl-vm11** への Bastion セッション内で、**[スタート]** をクリックし、**[スタート]** メニューで、**[リモート デスクトップ]** をクリックしてリモート デスクトップ クライアントを起動します。
1. **az140-cl-vm11** への Bastion セッション内で、**[リモート デスクトップ]** クライアント ウィンドウの **[サブスクライブ]** を選択し、プロンプトが表示されたら、**aduser1** の認証情報でサインインします。

   >**注** サブスクライブを求めらない場合は、以前のサブスクリプションの登録の解除が必要な場合があります。

1. アプリケーションの一覧で、**[コマンド プロンプト]** をダブルクリックし、プロンプトが表示されたら、**aduser1** アカウントのパスワードを入力し、**[コマンド プロンプト]** ウィンドウが正常に開くことを確認します。
1. **[コマンド プロンプト]** ウィンドウの左上隅で、**[コマンド プロンプト]** アイコンを右クリックし、ドロップダウン メニューの **[プロパティ]** を選択します。
1. **[コマンド プロンプトのプロパティ]** ダイアログ ボックスで、**[フォント]** タブを選択し、サイズとフォントの設定を変更して、**[OK]** を選択します。
1. **[コマンド プロンプト]** ウィンドウで、**「ログオフ」** と入力し、**Enter** キーを押してリモート デスクトップ セッションからサインアウトします。
1. **az140-cl-vm11** への Bastion セッション内で、**[リモート デスクトップ]** クライアント ウィンドウのアプリケーションの一覧で、[az140-21-ws1] の下にある **[SessionDesktop]** をダブルクリックし、リモート デスクトップ セッションが起動することを確認します。 
1. **[SessionDesktop]** セッション内で、**[スタート]** を右クリックし、右クリック メニューの **[実行]** を選択し、**[実行]** ダイアログ ボックスの **[開く]** テキスト ボックスに **「cmd」** と入力し、**[OK]** を選択して **[コマンド プロンプト]** ウィンドウを起動します。
1. **[コマンド プロンプト]** ウィンドウの設定が、このタスクで前に構成したものと一致することを確認します。
1. **[SessionDesktop]** セッション内で、すべてのウィンドウを最小化し、デスクトップを右クリックし、右クリック メニューで **[新規]** を選択し、カスケード メニューで **[ショートカット]** を選択します。 
1. **[ショートカットの作成]** ウィザードの **[どの項目にショートカットを作成しますか?]** ページで、**[項目の場所を入力してください]** テキスト ボックスに「**メモ帳**」と入力し、**[次へ]** を選択します。
1. **[ショートカットの作成]** ウィザードの **[ショートカットの名前を付ける]** ページで、**[このショートカットの名前を入力してください]** テキスト ボックスに「**メモ帳**」と入力し、**[完了]** を選択します。
1. **[SessionDesktop]** セッション内で、**[スタート]** を右クリックし、右クリック メニューで **[シャットダウンまたはサインアウト]** を選択し、カスケード メニューで **[サインアウト]** をクリックします。
1. **az140-cl-vm11** への Bastion セッションに戻り、**[リモート デスクトップ]** クライアント ウィンドウのアプリケーションの一覧で、**[SessionDesktop]** をダブルクリックし、新しいリモート デスクトップ セッションを開始します。 
1. **[SessionDesktop]** セッション内で、**メモ帳**ショートカットがデスクトップに表示されることを確認します。
1. **[SessionDesktop]** セッション内で、**[スタート]** を右クリックし、右クリック メニューで **[シャットダウンまたはサインアウト]** を選択し、カスケード メニューで **[サインアウト]** をクリックします。
1. ラボのコンピュータに切り替えて、Azure portal を表示している Microsoft Edge ウィンドウの **[ストレージ アカウント]** ブレードに移動し、前の演習で作成したストレージ アカウントを表すエントリを選択します。
1. [ストレージ アカウント] ブレードの **[ファイル サービス]** セクションで **[ファイル共有]** を選択し、ファイル共有の一覧で **[az140-22-profiles]** を選択します。 
1. **[az140-22-profiles]** ブレードで、**[参照]** を選択し、そのコンテンツに **ADATUM\\aaduser1** アカウントのセキュリティ識別子 (SID) とそれに続く **_aaduser1** サフィックスの組み合わせで構成される名前のフォルダーが含まれていることを確認します。
1. 前の手順で特定したフォルダーを選択し、**"Profile_aduser1.vhd"** という名前の 1 つのファイルが含まれていることに注意してください。

### 演習 2: ラボでプロビジョニングおよび使用されている Azure VM を停止して割り当てを解除する

この演習の主なタスクは次のとおりです。

1. ラボでプロビジョニングおよび使用されている Azure VM を停止して割り当てを解除する

>**注**: この演習では、関連するコンピューティング料金を最小限に抑えるために、このラボでプロビジョニングおよび使用されている Azure VM の割り当てを解除します

#### タスク 1: ラボでプロビジョニングおよび使用されている Azure VM の割り当てを解除する

1. ラボのコンピューターに切り替え、Azure portal が表示されている Web ブラウザー ウィンドウにおいて、**[Cloud Shell]** ペイン内で **[PowerShell]** シェル セッションを開きます。
1. [Cloud Shell] ウィンドウの PowerShell セッションから、次を実行して、このラボで作成および使用されているすべての Azure VM を一覧表示します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. [Cloud Shell] ウィンドウの PowerShell セッションから、次を実行して、このラボで作成および使用したすべての Azure VM を停止して割り当てを解除します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注**: このコマンドは非同期で実行されるため (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に停止されて割り当てが解除されるまでに数分かかります。
