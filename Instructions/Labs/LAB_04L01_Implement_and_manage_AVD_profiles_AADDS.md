---
lab:
  title: 'ラボ: Azure Virtual Desktop プロファイルを実装して管理する (Microsoft Entra AD DS)'
  module: 'Module 4: Manage User Environments and Apps'
---

# ラボ - Azure Virtual Desktop プロファイルを実装して管理する (Microsoft Entra DS)
# 受講生用ラボ マニュアル

## ラボの依存関係

- Azure サブスクリプション
- Azure サブスクリプションに関連付けられた Microsoft Entra テナントの全体管理者ロールと、Azure サブスクリプションの所有者または共同作成者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウント
- ラボでプロビジョニングした Azure Virtual Desktop 環境 **Azure Virtual Desktop (Microsoft Entra DS) の概要**

## 推定所要時間

30 分

## ラボのシナリオ

Microsoft Entra DS 環境で Azure Virtual Desktop プロファイル管理を実装する必要があります。

## 目標
  
このラボを完了すると、次のことができるようになります。

- Microsoft Entra DS 環境で Azure Virtual Desktop 用のプロファイル コンテナーを格納するように Azure Files を構成する
- Microsoft Entra DS 環境で Azure Virtual Desktop 用に FSLogix ベースのプロファイルを実装する

## ラボ ファイル

- なし

## 手順

### 演習 1: Azure Virtual Desktop 用に FSLogix ベースのプロファイルを実装する

この演習の主なタスクは次のとおりです。

1. Azure Virtual Desktop セッション ホスト VM でローカル管理者グループを構成する
1. Azure Virtual Desktop セッション ホスト VM で FSLogix ベースのプロファイルを構成する
1. Azure Virtual Desktop 用に FSLogix ベースのプロファイルをテストする
1. Azure ラボ リソースを削除する

#### タスク 1: Azure Virtual Desktop セッション ホスト VM でローカル管理者グループを構成する

1. ラボのコンピューターから Web ブラウザーを起動して [Azure portal](https://portal.azure.com) に移動し、このラボで使用しているサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。
1. Azure portal で、検索テキストボックスのすぐ右にあるツールバー アイコンを選択して **[Cloud Shell]** ペインを開きます。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、 **[PowerShell]** を選択します。 

   >**注**: **Cloud Shell** を初めて起動し、[**ストレージがマウントされていません**] というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、**[ストレージの作成]** を選択します。 

1. **[Cloud Shell]** ペインの PowerShell セッションから、次を実行して、このラボで使用する Azure Virtual Desktop セッション ホスト Azure VMを開始します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Start-AzVM
   ```

   >**注**: 次の手順に進む前に、Azure VM が実行されるまで待ちます。
   
      
1. **[Cloud Shell]** ペインの PowerShell セッションから、次を実行して、セッション ホストで PowerShell リモート処理を有効にします。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Enable-AzVMPSRemoting
   ```
   
1. Cloud Shell を閉じる
1. ラボ コンピューターの Azure portal で、「**仮想マシン**」を検索して選択し、**[仮想マシン]** ウィンドウで **[az140-cl-vm11a]** エントリを選択します。 これにより、[**az140-cl-vm11a**] ブレードが開きます。
1. **[az140-cl-vm11a]** ブレードで **[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、**[az140-cl-vm11a \| 接続]** ブレードの **[Bastion]** タブで **[Bastion を使用する]** を選択します。
1. プロンプトが表示されたら、次の資格情報を入力し、**[接続]** を選択します。

   |設定|値|
   |---|---|
   |ユーザー名|**aadadmin1@adatum.com**|
   |Password|以前に構成したパスワード|

1. **az140-cl-vm11a** への Bastion セッション内のスタート メニューで、**Windows 管理ツール** フォルダーに移動して展開し、**[Active Directory ユーザーとコンピューター]** を選択します。
1. **[Active Directory ユーザーとコンピューター]** コンソールで、ドメイン ノードを右クリックし、**[新規]**、**[組織単位]** の順に選択し、**[新しいオブジェクト - 組織単位]** ダイアログ ボックスの **[名前]** テキストボックスに「**ADDC ユーザー**」と入力して、**[OK]** を選択します。
1. **Active Directory ユーザーとコンピューター** コンソールで、**AD DC ユーザー**を右クリックし、**[新規]**、**[グループ]** の順に選択し、**[新しいオブジェクト-組織単位]** ダイアログ ボックスで、次の設定を指定して **[OK]** を選択します。

   |設定|値|
   |---|---|
   |グループ名|**ローカル管理者**|
   |グループ名 (Windows 2000 より前)|**ローカル管理者**|
   |Group scope|**Global**|
   |グループの種類|**Security**|

1. **[Active Directory ユーザーとコンピューター]** コンソールで、**[ローカル管理者]** グループのプロパティを表示し、**[メンバー]** タブに切り替え、**[追加]** を選択し、**[ユーザー、連絡先、コンピューター、サービス アカウント、またはグループの選択]** ダイアログ ボックスで、**[選択するオブジェクト名を入力]** で「**aadadmin1;wvdaadmin1**」と入力して、**[OK]** を選択します。
1. **az140-cl-vm11a** への Bastion セッション内のスタート メニューで、**Windows 管理ツール** フォルダーに移動して展開し、**グループ ポリシー管理**を選択します。
1. **グループ ポリシー管理**コンソールで、**AADDC コンピューター** OU に移動し、**AADDC コンピューター GPO** アイコンを右クリックし、**[編集]** を選択します。
1. **グループ ポリシー管理エディター** コンソールで、**[コンピューターの構成]**、**[ポリシー]**、**[Windows 設定]**、**[セキュリティ設定]** の順に展開し、**[制限されたグループ]** を右クリックし、**[グループの追加]** を選択します。
1. **[グループの追加]** ダイアログ ボックスの **[グループ]** テキスト ボックスで、**[参照]** を選択し、**[グループの選択]** ダイアログ ボックスで、**選択するオブジェクト名を**入力し、**「ローカル 管理」** と入力し、**[OK]** を選択します。
1. **[グループの追加]** ダイアログ ボックスに戻り、**[OK]** を選択します。
1. **[ADATUM\ Local Admins のプロパティ]** ダイアログ ボックスの **[このグループはメンバーです]** というラベルの付いたセクションで、**[追加]** を選択し、**[グループ メンバー シップ]** ダイアログ ボックスで「**Administrators**」と入力して **[OK]** を選択し、もう一度 **[OK]** を選択して変更を確定します。

   >**注**: **「このグループは次のメンバーである」** というラベルのセクションを必ず使用してください。

1. az140-cl-vm11a Azure VM への Bastion セッション内で、PowerShell ISE を管理者として起動し、次を実行して 2 つの Azure Virtual Desktop ホストを再起動して、グループ ポリシー処理をトリガーします。

   ```powershell
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Restart-Computer -ComputerName $servers -Force -Wait
   ```

1. スクリプトが完了するのを待ちます。 これには 3 分ほどかかります。

#### タスク 2: Azure Virtual Desktop セッション ホスト VM に FSLogix ベースのプロファイルを構成する

1. **az140-cl-vm11a** への Bastion セッション内で、**az140-21-p1-0** へのリモート デスクトップ セッションを開始し、プロンプトが表示されたら、**ADATUM\wvdaadmin1** のユーザー アカウントの作成時に設定したユーザー名とパスワードを入力してサインインします。 

   > **注**: RDP 接続を使用しても接続できない場合は、Azure portal で Bastion を使用して VM に接続します。

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、Microsoft Edge を起動して、[FSLogix ダウンロード ページ](https://aka.ms/fslogix_download)を参照し、FSLogix 圧縮インストール バイナリをダウンロードし、これを **C:\\Source** フォルダーに抽出して、**x64\\Release** サブフォルダーに移動し、**FSLogixAppsSetup.exe** を使用して、Microsoft FSLogix Apps を既定の設定でインストールします。

   > **注**: イメージに既に含まれているかどうかによって、FXLogic のインストールは必要ない場合があります。 FX Logix のインストールには再起動が必要です。

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、**Windows PowerShell ISE** を管理者として起動し、**[Administrator: Windows PowerShell ISE]** スクリプト ペインから、次を実行して、最新バージョンの PowerShellGet モジュールをインストールします (確認を求められたら、**[はい]** を選択します)。

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールから、以下を実行して、最新バージョンの Az PowerShell モジュールをインストールします (確認を求められたら、**[すべてはい]** を選択します)。

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. **[Administrator: Windows PowerShell ISE]** コンソールで、次を実行して、実行ポリシーを変更します。

   ```powershell
   Set-ExecutionPolicy RemoteSigned -Force
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールから、以下を実行して、Azure サブスクリプションにサインインします。

   ```powershell
   Connect-AzAccount
   ```

1. プロンプトが表示されたら、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの Microsoft Entra 資格情報を指定してサインインします。
1. **[Administrator: Windows PowerShell ISE]** スクリプト ペインで、次を実行して、このラボで以前に構成した Azure ストレージ アカウントの名前を取得します。

   ```powershell
   $resourceGroupName = 'az140-22a-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. **az140-21-p1-0** へのリモート デスクトップ セッション内の **[Administrator: Windows PowerShell ISE]** スクリプト ペインで、次を実行して、プロファイル レジストリ設定を構成します。

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```
   >**注** コマンドを実行したらエラーが発生した場合は、次の手順に進んでください。
   
1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、**[スタート]** を右クリックし、右クリック メニューの **[実行]** を選択し、**[実行]** ダイアログ ボックスの **[開く]** テキスト ボックスに次のように入力し、**[OK]** を選択して **[ローカル ユーザーとグループ]** ウィンドウを起動します。

   ```cmd
   lusrmgr.msc
   ```

1. **ローカル ユーザーとグループ** コンソールで、**FSLogix** 文字列で始まる名前の 4 つのグループに注意してください。

   - FSLogix ODFC 除外一覧
   - FSLogix ODFC 対象一覧
   - FSLogix プロファイル除外一覧
   - FSLogix プロファイル対象一覧

1. **[ローカル ユーザーとグループ]** コンソールで、**[FSLogix プロファイ包含リスト]** グループをダブルクリックし、**\\Everyone** グループが含まれていることを確認し、**[OK]** を選択して、グループの **[プロパティ]** ウィンドウを閉じます。 
1. [**ローカル ユーザーとグループ**] コンソールで、**[FSLogix プロファイルの除外リスト]** グループ エントリをダブルクリックし (既定では、グループ メンバーは含まれていません)、**[OK]** を選択して、グループの **[プロパティ]** ウィンドウを閉じます。 

   > **注**: 一貫性のあるユーザー エクスペリエンスを提供するには、すべての Azure Virtual Desktop セッション ホストに FSLogix コンポーネントをインストールして構成する必要があります。 このタスクは、ラボ環境の他のセッション ホストで無人で実行します。 

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、**[Administrator: Windows PowerShell ISE]** スクリプト ペインから、次を実行し、**az140-21-p1-1** セッション ホストに FSLogix コンポーネントをインストールします。

   ```powershell
   $server = 'az140-21-p1-1' 
   $localPath = 'C:\Source\x64'
   $remotePath = "\\$server\C$\Source\x64\Release"
   Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
   Invoke-Command -ComputerName $server -ScriptBlock {
      Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
   } 
   ```

1. **az140-21-p1-0** へのリモート デスクトップ セッション内で、**Windows PowerShell ISE** を管理者として起動し、**[Administrator: Windows PowerShell ISE]** スクリプト ペインから次を実行して、**az140-21-p1-1** セッション ホストでプロファイル レジストリ設定を構成します。

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   Invoke-Command -ComputerName $server -ScriptBlock {
      New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$using:fileShareName"
   }
   ```

   > **注**: FSLogix ベースのプロファイル機能をテストする前に、テストに使用する ADATUM\wvdaadmin1 アカウントの、ローカルにキャッシュされたプロファイルを、前のラボで使用した Azure Virtual Desktop セッション ホストから削除する必要があります。

1. Bastion セッションを **az140-cl-vm11a** に切り替え、Bastion セッション内の **az140-cl-vm11a** で、**[Administrator: Windows PowerShell ISE]** ウィンドウに切り替え、**[Administrator: Windows PowerShell ISE]** スクリプト ペインから、以下を実行して ADATUM\aaduser1 アカウントのローカルにキャッシュされたプロファイルを削除します。

   ```powershell
   $userName = 'aaduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### タスク 3: Azure Virtual Desktop を使用して FSLogix ベースのプロファイルをテストする

1. **az140-cl-vm11a** への Bastion セッション内で、リモート デスクトップ クライアントに切り替えます。
1. **az140-cl-vm11** への Bastion セッション内で、**[リモート デスクトップ]** クライアント ウィンドウのアプリケーションのリスト内にある **[コマンド プロンプト]** をダブルクリックし、プロンプトが表示されたらパスワードを入力し、**[コマンド プロンプト]** ウィンドウが起動することを確認します。 

   > **注**: 最初はアプリケーションが起動するまでに数分かかる場合がありますが、その後、アプリケーションの起動は大幅に速くなります。

1. **[コマンド プロンプト]** ウィンドウの左上隅で、**[コマンド プロンプト]** アイコンを右クリックし、ドロップダウン メニューの **[プロパティ]** を選択します。
1. **[コマンド プロンプトのプロパティ]** ダイアログ ボックスで、**[フォント]** タブを選択し、サイズとフォントの設定を変更して、**[OK]** を選択します。
1. **[コマンド プロンプト]** ウィンドウで、**「ログオフ」** と入力し、**Enter** キーを押してリモート デスクトップ セッションからサインアウトします。
1. **az140-cl-vm11** への Bastion セッション内で、[**リモート デスクトップ**] クライアント ウィンドウのアプリケーションのリスト内にある [**SessionDesktop**] をダブルクリックし、リモート デスクトップ セッションが起動することを確認します。 
1. **[SessionDesktop]** セッション内で、**[スタート]** を右クリックし、右クリック メニューの **[実行]** を選択し、**[実行]** ダイアログ ボックスの **[開く]** テキスト ボックスに **「cmd」** と入力し、**[OK]** を選択して **[コマンド プロンプト]** ウィンドウを起動します。
1. **[コマンド プロンプト]** ウィンドウのプロパティが、このタスクで前に設定したものと一致することを確認します。
1. **[SessionDesktop]** セッション内で、すべてのウィンドウを最小化し、デスクトップを右クリックし、右クリック メニューで **[新規]** を選択し、カスケード メニューで **[ショートカット]** を選択します。 
1. **[ショートカットの作成]** ウィザードの **[どの項目にショートカットを作成しますか?]** ページで、**[項目の場所を入力してください]** テキスト ボックスに「**メモ帳**」と入力し、**[次へ]** を選択します。
1. **[ショートカットの作成]** ウィザードの **[ショートカットの名前を付ける]** ページで、**[このショートカットの名前を入力してください]** テキスト ボックスに「**メモ帳**」と入力し、**[完了]** を選択します。
1. **[SessionDesktop]** セッション内で、**[スタート]** を右クリックし、右クリック メニューで **[シャットダウンまたはサインアウト]** を選択し、カスケード メニューで **[サインアウト]** をクリックします。
1. **az140-cl-vm11** への Bastion セッションに戻り、**[リモート デスクトップ]** クライアント ウィンドウのアプリケーションの一覧で、**[SessionDesktop]** をダブルクリックし、新しいリモート デスクトップ セッションを開始します。 
1. **[SessionDesktop]** セッション内で、**メモ帳**ショートカットがデスクトップに表示されることを確認します。
1. **[SessionDesktop]** セッション内で、**[スタート]** を右クリックし、右クリック メニューで **[シャットダウンまたはサインアウト]** を選択し、カスケード メニューで **[サインアウト]** をクリックします。
1. Bastion セッションを **az140-cl-vm11a** に切り替え、Azure portal を表示している [Microsoft Edge] ウィンドウに切り替えます。
1. Azure portal を表示している [Microsoft Edge] ウィンドウの **[ストレージ アカウント]** ブレードに戻り、前の演習で作成したストレージ アカウントを表すエントリを選択します。
1. [ストレージ アカウント] ブレードの **[ファイル サービス]** セクションで **[ファイル共有]** を選択し、ファイル共有の一覧で **az140-22a-profiles** を選択します。 
1. **[az140-22a-profiles]** ブレードで、**[参照]** を選択し、そのコンテンツに **ADATUM\\aaduser1** アカウントのセキュリティ識別子 (SID) とそれに続く **_aaduser1** サフィックスの組み合わせで構成される名前のフォルダーが含まれていることを確認します。
1. 前の手順で特定したフォルダーを選択し、**「Profile_aduser1.vhd」** という名前の 1 つのファイルが含まれていることに注意してください。

### 演習 2: Azure のラボ リソースを削除する (省略可能)

1. [「Azure portal を使用して Azure Active Directory Domain Services マネージド ドメインを削除する」]( https://docs.microsoft.com/en-us/azure/active-directory-domain-services/delete-aadds)で説明されている手順に従って、Microsoft Entra DS のデプロイを削除します。
1. [「Azure Resource Manager リソース グループとリソースの削除」](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/delete-resource-group?tabs=azure-portal)で説明されている手順に従って、このコースの Microsoft Entra DS ラボでプロビジョニングしたすべての Azure のリソース グループを削除します。
