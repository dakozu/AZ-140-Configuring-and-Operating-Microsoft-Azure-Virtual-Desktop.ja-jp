---
lab:
  title: ラボ:Azure Virtual Desktop アプリケーション (AD DS) のパッケージ化
  module: 'Module 4: Manage User Environments and Apps'
ms.openlocfilehash: 936e56ddbadf1f0b26e6bc8802018f332befc540
ms.sourcegitcommit: 040b38677ab4bfb0bf430a8797aaa8f524dc501c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/07/2022
ms.locfileid: "145883986"
---
# <a name="lab---package-azure-virtual-desktop-applications-ad-ds"></a>ラボ - Azure Virtual Desktop アプリケーション (AD DS) のパッケージ化
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-dependencies"></a>ラボの依存関係

- Azure サブスクリプション
- Azure サブスクリプションに関連付けられている Azure AD テナントでグローバル管理者ロールになっていて、Azure サブスクリプションで所有者または共同作成者のロールになっている、Microsoft アカウントまたは Azure AD アカウント
- 実施するラボ - **Azure Virtual Desktop (AD DS) のデプロイを準備する**
- 実施するラボ - **Azure Virtual Desktop プロファイルの管理 (AD DS)**
- 実施するラボ - **WVD (AD DS) に対する条件付きアクセス ポリシーを構成する**

## <a name="estimated-time"></a>推定所要時間

約 60 分

## <a name="lab-scenario"></a>ラボのシナリオ

Azure Virtual Desktop アプリケーションを Active Directory ドメイン サービス (AD DS) 環境にパッケージ化してデプロイする必要があります。

## <a name="objectives"></a>目標
  
このラボを完了すると、次のことができるようになります。

- MSIX アプリ パッケージを準備して作成する
- Azure AD DS 環境に Azure Virtual Desktop 用 MSIX アプリのアタッチ イメージを実装する
- AD DS 環境の Azure Virtual Desktop に MSIX アプリのアタッチを実装する

## <a name="lab-files"></a>ラボ ファイル 

-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json
-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json

## <a name="instructions"></a>手順

### <a name="exercise-1-prepare-for-and-create-msix-app-packages"></a>演習 1:MSIX アプリ パッケージを準備および作成する

この演習の主なタスクは次のとおりです。

1. Azure Virtual Desktop セッション ホストの構成を準備する
1. Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする
1. Windows 10 を実行している Azure VM を MSIX パッケージ用に準備する
1. 署名証明書を生成する
1. パッケージ化するソフトウェアをダウンロードする
1. MSIX パッケージ ツールをインストールする
1. MSIX パッケージを作成する

#### <a name="task-1-prepare-for-configuration-of-azure-virtual-desktop-session-hosts"></a>タスク 1:Azure Virtual Desktop セッション ホストの構成を準備する

1. ラボのコンピューターから Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者の役割を持つユーザーアカウントの認証情報を提供してサインインします。
1. ラボ コンピューターにの Azure portal を表示している Web ブラウザーの画面で、**Cloud Shell** ペイン内の **PowerShell** シェル セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで使用する Azure VM をホストする Azure Virtual Desktop セッションを開始します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**注**:コマンドは非同期で実行されるので (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に開始されるまでに数分かかります。 

   >**注**:Azure VM が起動するのを待たずに、次のタスクに直接進みます。

#### <a name="task-2-deploy-an-azure-vm-running-windows-10-by-using-an-azure-resource-manager-quickstart-template"></a>タスク 2:Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする

1. ラボ コンピューターから、Azure portal を表示している Web ブラウザー ウィンドウの Cloud Shell ペインのツールバーで、 **[ファイルのアップロード/ダウンロード]** アイコンを選択し、ドロップダウン メニューで **[アップロード]** を選択し、 **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json** ファイルと **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json** ファイルを Cloud Shell ホーム ディレクトリにアップロードします。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、MSIX パッケージの作成に使用する Windows 10 を実行している Azure VM をデプロイし、Azure AD DS ドメインに参加させます。

   ```powershell
   $vNetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vNetResourceGroupName).Location
   $resourceGroupName = 'az140-42-RG'
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0402vmDeployment `
     -TemplateFile $HOME/az140-42_azuredeploycl42.json `
     -TemplateParameterFile $HOME/az140-42_azuredeploycl42.parameters.json
   ```

   > **注**: デプロイが完了するまで待ってから、次のタスクに進んでください。 これには 10 分ほどかかる場合があります。 

#### <a name="task-3-prepare-the-azure-vm-running-windows-10-for-msix-packaging"></a>タスク 3:Windows 10 を実行している Azure VM を MSIX パッケージ用に準備する

1. ラボ コンピューターの Azure portal で、「**仮想マシン**」を検索して選択し、 **[仮想マシン]** ウィンドウの仮想マシンの一覧で **[az140-cl-vm42]** エントリを選択します。 これにより、 **[az140-cl-vm42]** ウィンドウが開きます。
1. **[az140-cl-vm42]** ウィンドウで **[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、 **[az140-cl-vm42 \| 接続]** ウィンドウの **[Bastion]** タブで **[Bastion を使用する]** を選択します。
1. プロンプトが表示されたら、ユーザー名 **wvdadmin1** と、このユーザー アカウントを作成したときに設定したパスワードを使用してサインインします。 
1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、管理者として **Windows PowerShell ISE** を起動し、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、MSIX パッケージ用にオペレーティング システムを準備します。

   ```powershell
   Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
   reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
   reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
   reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
   ```

   > **注**:これらのレジストリの最後の変更により、ユーザー アクセス制御が無効になります。 これは技術的には必須ではありませんが、このラボで説明するプロセスを簡素化します。

#### <a name="task-4-generate-a-signing-certificate"></a>タスク 4:署名証明書を生成する

> **注**:このラボでは、自己署名証明書を使用します。 実稼働環境では、使用目的に応じて、公的認証局または内部認証局のいずれかによって発行された証明書を使用する必要があります。

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、共通名属性が **Adatum** に設定された自己署名証明書を生成し、**ローカル マシン** 証明書ストアの "**個人用**" フォルダーに証明書を保存します。

   ```powershell
   New-SelfSignedCertificate -Type Custom -Subject "CN=Adatum" -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -CertStoreLocation "cert:\LocalMachine\My"
   ```

1. **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、ローカル マシン証明書ストアを対象とする **[証明書]** コンソールを起動します。

   ```powershell
   certlm.msc
   ```

1. **[証明書]** コンソール ペインで、 **[個人用]** フォルダーを展開し、 **[証明書]** サブフォルダーを選択して、**Adatum** 証明書を右クリックし、右クリック メニューで **[すべてのタスク]** 、 **[エクスポート]** の順に選択します。 これにより、**証明書エクスポート ウィザード** が起動します。 
1. **証明書エクスポート ウィザード** の **[証明書エクスポート ウィザードへようこそ]** ページで、 **[次へ]** を選択します。
1. **証明書エクスポート ウィザード** の **[秘密鍵のエクスポート]** ページで、 **[はい、秘密鍵をエクスポートします]** オプションを選択し、 **[次へ]** を選択します。
1. **[証明書エクスポート] ウィザード** の **[ファイル形式のエクスポート]** ページで、 **[すべての拡張プロパティをエクスポートする]** チェックボックスをオンにし、 **[証明書のプライバシーを有効にする]** チェックボックスをオフにして、 **[次へ]** を選択します。
1. **[証明書エクスポート]** ウィザードの **[セキュリティ]** ページで、 **[パスワード]** チェックボックスをオンにし、下のテキストボックスに「**Pa55w.rd1234**」と入力して、 **[次へ]** を選択します。
1. **[証明書エクスポート]** ウィザードの **[エクスポートするファイル]** ページの **[ファイル名]** テキストボックスで **[参照]** を選択し、 **[名前を付けて保存]** ダイアログ ボックスで **C:\\Allfiles\\Labs\\04** フォルダーに移動し (最初にフォルダーを作成します)、 **[ファイル名]** テキストボックスに「**adatum.pfx**」と入力し、 **[保存]** を選択します。
1. **[証明書エクスポート]** ウィザードの **[エクスポートするファイル]** ページに戻り、テキストボックスにエントリ "**C:\\Allfiles\\Labs\\04\\adatum.pfx**" が含まれていることを確認して、 **[次へ]** を選択します。
1. **[証明書エクスポート] ウィザード** の **[証明書エクスポートウィザードの完了]** ページで、 **[完了]** を選択し、 **[OK]** を選択してエクスポートが成功したことを確認します。 

   > **注**:自己署名証明書を使用しているため、ターゲット セッション ホストの **Trusted People** 証明書ストアにインストールする必要があります。

1. **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、新しく生成された証明書をターゲット セッション ホストの **信頼できるユーザー** 証明書ストアにインストールします。

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   $cleartextPassword = 'Pa55w.rd1234'
   $securePassword = ConvertTo-SecureString $cleartextPassword -AsPlainText -Force
   $localPath = 'C:\Allfiles\Labs\04'
   ForEach ($wvdhost in $wvdhosts){
      $remotePath = "\\$wvdhost\C$\Allfiles\Labs\04\"
      New-Item -ItemType Directory -Path $remotePath -Force
      Copy-Item -Path "$localPath\adatum.pfx" -Destination $remotePath -Force
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Import-PFXCertificate -CertStoreLocation Cert:\LocalMachine\TrustedPeople -FilePath 'C:\Allfiles\Labs\04\adatum.pfx' -Password $using:securePassword
      } 
   }
   ```

#### <a name="task-5-download-software-to-package"></a>タスク 5:パッケージ化するソフトウェアをダウンロードする

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、**Microsoft Edge** を起動し、 **https://github.com/microsoft/XmlNotepad** を参照します。
1. **[microsoft/XmlNotepad** **readme.md]** ページで、ダウンロード可能なスタンドアロン インストーラーのダウンロード リンクを選択し、圧縮されたインストール ファイルをダウンロードします。
1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、ファイル エクスプローラーを起動し、"**ダウンロード**" フォルダーに移動し、圧縮ファイルを開き、そのコンテンツをコピーして、**C:\\AllFiles\\Labs\\04\\** ディレクトリに貼り付けます。 

#### <a name="task-6-install-the-msix-packaging-tool"></a>タスク 6:MSIX パッケージ ツールをインストールする

1. **az140-cl-vm42** へのリモート デスクトップセッション内で、**Microsoft Store** アプリを起動します。
1. **Microsoft Store** アプリで、**MSIX パッケージ ツール** を検索して選択し、 **[MSIX パッケージ ツール]** ページで **[取得]** を選択します。
1. プロンプトが表示されたら、サインインをスキップし、インストールが完了するのを待って **[開く]** を選択し、 **[診断データの送信]** ダイアログ ボックスで **[拒否]** を選択します。 

#### <a name="task-7-create-an-msix-package"></a>タスク 7:MSIX パッケージを作成する

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** ウィンドウに切り替え、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、Windows Search サービスを無効にします。

   ```powershell
   $serviceName = 'wsearch'
   Set-Service -Name $serviceName -StartupType Disabled
   Stop-Service -Name $serviceName
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、MSIX パッケージをホストするフォルダーを作成します。

   ```powershell
   New-Item -ItemType Directory -Path 'C:\AllFiles\Labs\04\XmlNotepad' -Force
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、抽出されたインストーラ ファイルから Zone.Identifier 代替データ ストリームを削除します。これには、インターネットからダウンロードされたことを示す値 "3" が含まれています。

   ```powershell
   Get-ChildItem -Path 'C:\AllFiles\Labs\04' -Recurse -File | Unblock-File
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、**MSIX パッケージ ツール インターフェイス** に切り替え、 **[タスクの選択]** ページで **[アプリケーション パッケージ] - [アプリ パッケージ エントリの作成]** を選択します。 これにより、**新しいパッケージの作成** ウィザードが起動します。
1. **[新しいパッケージの作成]** ウィザードの **[環境の選択]** ページで、 **[このコンピューターにパッケージを作成する]** オプションが選択されていることを確認し、 **[次へ]** を選択して、**MSIX パッケージ作成ツールドライバー** のインストールを待ちます。
1. **[新しいパッケージの作成]** ウィザードの **[コンピューターの準備]** ページで、推奨事項を確認します。 保留中の再起動がある場合は、オペレーティング システムを再起動し、**ADATUM\wvdadmin1** アカウントを使用して、再度サインインし、**MSIX パッケージ ツール** を再起動してから続行します。 

   >**注**:MSIX パッケージ ツールは、Windows Update と Windows Search を一時的に無効にします。 この場合、Windows Search サービスはすでに無効になっています。 

1. **[新しいパッケージの作成]** ウィザードの **[コンピューターの準備]** ページで、 **[次へ]** をクリックします。
1. **[新しいパッケージの作成]** ウィザードの **[インストーラーの選択]** ページで、 **[パッケージ化するインストーラーの選択]** テキスト ボックスの横にある **[参照]** を選択し、 **[開く]** ダイアログ ボックスで **C:\\AllFiles\\Labs\\04** フォルダーを参照し、**XmlNotepadSetup.msi** を選択して、 **[開く]** をクリックします。 
1. **[新しいパッケージの作成]** ウィザードの **[インストーラーの選択]** ページの **[署名の設定]** ドロップダウン リストで、 **[証明書 (.pfx) を使用して署名する]** エントリを選択し、 **[証明書の参照]** テキストボックスの横にある **[参照]** を選択して、 **[開く]** ダイアログ ボックスで、**C:\\AllFiles\\Labs\\04** フォルダーに移動し、**adatum.pfx** ファイルを選択して、 **[開く]** をクリックし、 **[パスワード]** テキスト ボックスに 「**Pa55w.rd1234**」と入力して、 **[次へ]** を選択します。
1. **[新しいパッケージの作成]** ウィザードの **[パッケージ情報]** ページで、パッケージ情報を確認し、発行元名が **CN=Adatum** に設定されていることを確認して、 **[次へ]** を選択します。 これにより、ダウンロードしたソフトウェアのインストールがトリガーされます。
1. **[XML Notepad セットアップ]** ウィンドウで、使用許諾契約の条項に同意して **[インストール]** を選択し、インストールが完了したら、 **[XML Notepad の起動]** チェックボックスを選択して **[完了]** を選択します。
1. **[XML Notepad 分析]** ウィンドウで、プロンプトが表示されたら、 **[いいえ]** を選択し、XML Notepad が実行されていることを確認して閉じ、 **[MSIX パッケージ ツール]** ウィンドウで **[新しいパッケージの作成]** ウィザードに戻り、 **[次へ]** を選択します。

   > **注**:この場合、インストールを完了するために再起動する必要はありません。

1. **[新しいパッケージの作成]** ウィザードの **[最初の起動タスク]** ページで、提供された情報を確認し、 **[次へ]** を選択します。
1. "**完了しましたか?** " というプロンプトが表示されたら、 **[はい、次に進む]** を選択します。
1. **[新しいパッケージの作成]** ウィザードの **[サービス レポート]** ページで、サービスがリストされていないことを確認し、 **[次へ]** を選択します。
1. **[新しいパッケージの作成]** ウィザードの **[パッケージの作成]** ページの **[保存場所]** テキストボックスに、「**C:\\Allfiles\\Labs\\04\\XmlNotepad\XmlNotepad.msix**」と入力し、 **[作成]** をクリックします。
1. **[パッケージが正常に作成されました]** ダイアログ ボックスで、保存されたパッケージの場所をメモし、 **[閉じる]** を選択します。
1. ファイル エクスプローラー ウィンドウに切り替え、**C:\\Allfiles\\Labs\\04\\XmlNotepad** フォルダーに移動し、*.msix ファイルと *.xml ファイルが含まれていることを確認します。
1. **XmlNotepad.msix** ファイルを **C:\\Allfiles\\Labs\\04** フォルダーにコピーします。


### <a name="exercise-2-implement-an-msix-app-attach-image-for-azure-virtual-desktop-in-azure-ad-ds-environment"></a>演習 2:Azure AD DS 環境に Azure Virtual Desktop 用 MSIX アプリのアタッチ イメージを実装する

この演習の主なタスクは次のとおりです。

1. Window 10 Enterprise Edition を実行している Azure VM で Hyper-V を有効にする
1. MSIXアプリ アタッチ イメージを作成する

#### <a name="task-1-enable-hyper-v-on-the-azure-vms-running-window-10-enterprise-edition"></a>タスク 1:Window 10 Enterprise Edition を実行している Azure VM で Hyper-V を有効にする

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、MSIX アプリ アタッチ用にターゲット Azure Virtual Desktop ホストを準備します。 

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
         reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
         reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
         reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
         Set-Service -Name wuauserv -StartupType Disabled
      }
   }
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから次のコマンドを実行して、Hyper-V とその管理ツール (Hyper-V PowerShell モジュールを含む) を Azure Virtual Desktop ホストにインストールします。

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
      }
   }
   ```

1. ターゲット オペレーティング システムを起動するかどうかを尋ねられたら、 **[はい]** を選択します。
1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、Hyper-V とその管理ツール (Hyper-V PowerShell モジュールを含む) をローカル コンピューターにインストールします。

   ```powershell
   Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
   ```

1. Hyper-V コンポーネントのインストールが完了したら、 **[はい]** を選択してオペレーティング システムを再起動します。 再起動後、アカウント **wvdadmin1** とパスワード **Pa55w.rd1234** を使用してもう一度サインインします。

#### <a name="task-2-create-an-msix-app-attach-image"></a>タスク 2:MSIXアプリ アタッチ イメージを作成する

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、**Microsoft Edge** を起動し、 **https://aka.ms/msixmgr** を参照します。 これにより、**msixmgr.zip** ファイル (MSIX mgr ツール アーカイブ) が "**ダウンロード**" フォルダーに自動的にダウンロードされます。
1. ファイル エクスプローラーで、"**ダウンロード**" フォルダーに移動し、圧縮ファイルを開いて、**x64** フォルダーの内容を **C:\\AllFiles\\Labs\\04** フォルダーにコピーします。 
1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、管理者として **Windows PowerShell ISE** を起動し、 **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、MSIX アプリ アタッチ イメージとして機能する VHD ファイルを作成します。

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Allfiles\Labs\04\MSIXVhds' -Force
   New-VHD -SizeBytes 128MB -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Dynamic -Confirm:$false
   ```

1. **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、新しく作成された VHD ファイルをマウントします。

   ```powershell
   $vhdObject = Mount-VHD -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Passthru
   ```

1. **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行してディスクを初期化し、新しいパーティションを作成してフォーマットし、使用可能な最初のドライブ文字を割り当てます。

   ```powershell
   $disk = Initialize-Disk -Passthru -Number $vhdObject.Number
   $partition = New-Partition -AssignDriveLetter -UseMaximumSize -DiskNumber $disk.Number
   Format-Volume -FileSystem NTFS -Confirm:$false -DriveLetter $partition.DriveLetter -Force
   ```

   > **注**:F: ドライブのフォーマットを求めるポップアップ ウィンドウが表示されたら、 **[キャンセル]** を選択します。

1. **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、MSIX ファイルをホストするフォルダー構造を作成し、前のタスクで作成した MSIX パッケージに解凍します。

   ```powershell
   $appName = 'XmlNotepad'
   New-Item -ItemType Directory -Path "$($partition.DriveLetter):\Apps" -Force
   Set-Location -Path 'C:\AllFiles\Labs\04\x64'
   .\msixmgr.exe -Unpack -packagePath ..\$appName.msix -destination "$($partition.DriveLetter):\Apps" -applyacls
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、ファイル エクスプローラーで、**F:\\Apps** フォルダーに移動し、その内容を確認します。 このフォルダーへのアクセスを取得するように求められたら、 **[続行]** を選択します。
1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、MSIX イメージとして機能する VHD ファイルをマウント解除します。

   ```powershell
   Dismount-VHD -Path "C:\Allfiles\Labs\04\MSIXVhds\$appName.vhd" -Confirm:$false
   ```

### <a name="exercise-3-implement-msix-app-attach-on-azure-virtual-desktop-session-hosts"></a>演習 3:Azure Virtual Desktop セッションホストに MSIX アプリのアタッチを実装する

この演習の主なタスクは次のとおりです。

1. Azure Virtual Desktop ホストを含む Active Directory グループを構成する
1. MSIX アプリのアタッチの Azure Files 共有を設定する
1. MSIX アプリのアタッチ イメージを Azure Virtual Desktop セッションホストにマウントして登録する
1. MSIX アプリをアプリケーション グループに公開する
1. MSIX アプリのアタッチ機能を検証する

#### <a name="task-1-configure-active-directory-groups-containing-azure-virtual-desktop-hosts"></a>タスク 1:Azure Virtual Desktop ホストを含む Active Directory グループを構成する

1. ラボ コンピューターに切り替え、Azure portal を表示する Web ブラウザーで、「**仮想マシン**」を検索して選択し、 **[仮想マシン]** ウィンドウから **[az140-dc-vm11]** を選択します。
1. **[az140-dc-vm11]** ウィンドウで **[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、 **[az140-dc-vm11 \| 接続]** ウィンドウの **[Bastion]** タブで **[Bastion を使用する]** を選択します。
1. プロンプトが表示されたら、次の資格情報を入力し、 **[接続]** を選択します。

   |設定|値|
   |---|---|
   |[ユーザー名]|**学生**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、**Windows PowerShell ISE** を管理者として起動します。
1. **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、このラボで使用する Azure AD テナントに同期される AD DS グループ オブジェクトを作成します。

   ```powershell
   $ouPath = "OU=WVDInfra,DC=adatum,DC=com"
   New-ADGroup -Name 'az140-hosts-42-p1' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

   > **注**:このグループを使用して、Azure Virtual Desktop ホストに **az140-42-msixvhds** ファイル共有へのアクセス許可を付与します。

1. **[管理者: Windows PowerShell ISE]** コンソールから以下を実行して、前の手順で作成したグループにメンバーを追加します。

   ```powershell
   Get-ADGroup -Identity 'az140-hosts-42-p1' | Add-AdGroupMember -Members 'az140-21-p1-0$','az140-21-p1-1$','az140-21-p1-2$'
   ```

1. **[管理者: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、'az140-hosts-42-p1' グループのメンバーであるサーバーを再起動します。

   ```powershell
   $hosts = (Get-ADGroup -Identity 'az140-hosts-42-p1' | Get-ADGroupMember | Select-Object Name).Name
   $hosts | ForEach-Object {Restart-Computer -ComputerName $_ -Force}
   ```

   > **注**:この手順により、グループ メンバーシップの変更が確実に有効になります。 

1. **az140-dc-vm11** へのリモート デスクトップ セッション内の **[スタート]** メニューで、 **[Azure AD Connect]** フォルダーを展開し、 **[Azure AD Connect]** を選択します。
1. **[Microsoft Azure Active Directory Connect]** ウィンドウの **[Azure AD Connect へようこそ]** ページで、 **[構成]** を選択します。
1. **[Microsoft Azure Active Directory Connect]** ウィンドウの **[追加のタスク]** ページで、 **[同期オプションのカスタマイズ]** を選択し、 **[次へ]** を選択します。
1. **[Microsoft Azure Active Directory Connect]** ウィンドウの **[Azure AD に接続]** ページで、このタスクで先ほど特定した **aadsyncuser** ユーザー アカウントのユーザー プリンシパル名と、このユーザー アカウントの作成時に設定したパスワードを使用して認証します。
1. **[Microsoft Azure Active Directory Connect]** ウィンドウの **[ディレクトリの接続]** ページで、 **[次へ]** を選択します。
1. **[Microsoft Azure Active Directory Connect]** ウィンドウの **[ドメインと OU のフィルタリング]** ページで、 **[選択したドメインと OU を同期する]** オプションが選択されていることを確認し、**adatum.com** ノードを展開し、ToSync OU の横にあるチェックボックスが選択されていることを確認し、**WVDInfra** OU の横にあるチェックボックスを選択して (他の選択したチェックボックスは変更しないでください)、 **[次へ]** を選択します。
1. **[Microsoft Azure Active Directory Connect]** ウィンドウの **[オプション機能]** ページで、既定の設定を受け入れて、 **[次へ]** を選択します。
1. **[Microsoft Azure Active Directory Connect]** ウィンドウの **[構成の準備完了]** ページで、 **[構成の完了時に同期プロセスを開始する]** チェックボックスがオンになっていることを確認し、 **[構成]** を選択します。
1. **[構成が完了しました]** ページの情報を確認し、 **[終了]** を選択して **[Microsoft Azure Active Directory Connect]** ウィンドウを閉じます。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Microsoft Edge を起動して、[Azure portal](https://portal.azure.com) に移動します。 プロンプトが表示されたら、このラボで使用している Azure サブスクリプションに関連付けられている Azure AD テナントのグローバル管理者ロールを持つユーザーアカウントの Azure AD の資格情報を使用してサインインします。
1. **az140-dc-vm11** へのリモート デスクトップ セッション内で、Azure portal を表示している Microsoft Edge ウィンドウで、**Azure Active Directory** を検索して選択し、このラボで使用している Azure サブスクリプションに関連付けられている Azure AD テナントに移動します。
1. Azure Active Directory ブレードの左側の垂直メニューバーの **[管理]** セクションで、 **[グループ]** をクリックします。 
1. **[グループ | すべてのグループ]** ウィンドウで、グループの一覧から **[az140-hosts-42-p1]** エントリを選択します。

   > **注**:グループを表示するために、ページの更新が必要となる場合があります。

1. **[az140-hosts-42-p1]** ブレードの左側にある垂直メニュー バーの **[管理]** セクションで、 **[メンバー]** をクリックします。
1. **[az140-hosts-42-p1 | メンバー]** ウィンドウで、 **[ダイレクト メンバー]** の一覧に、このタスクで先ほどグループに追加した Azure Virtual Desktop プールの 3 つのホストが含まれていることを確認します。

#### <a name="task-2-set-up-the-azure-files-share-for-msix-app-attach"></a>タスク 2:MSIX アプリのアタッチの Azure Files 共有を設定する

1. ラボ コンピューターで、リモート デスクトップ セッションに戻って **az140-cl-vm42** に切り替えます。
1. **az140-cl-vm42** へのリモートデスクトップセッション内で、InPrivate モードで Microsoft Edge を起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者の役割を持つユーザー アカウントの認証情報を提供してサインインします。

   > **注**:必ず Microsoft Edge InPrivate モードを使用してください。

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、Azure portal を表示する Microsoft Edge ウィンドウで **ストレージ アカウント** を検索して選択し、 **[ストレージ アカウント]** ブレードで、ユーザー プロファイルをホストするように構成したストレージ アカウントを選択します。

   > **注**:ラボのこの部分は、ラボ - **Azure Virtual Desktop プロファイルの管理 (AD DS)** または **Azure Virtual Desktop プロファイルの管理 (Azure AD DS)** の完了を条件としています

   > **注**:実稼働シナリオでは、別のストレージ アカウントの使用を検討する必要があります。 これには、Azure AD DS 認証用にそのストレージ アカウントを構成する必要があります。これは、ユーザー プロファイルをホストするストレー ジアカウント用に既に実装されています。 同じストレージ アカウントを使用して、個々のラボ間で重複する手順を最小限に抑えています。

1. [ストレージ アカウント] ウィンドウの左側の垂直メニューで、 **[アクセス制御 (IAM)]** を選択します。
1. ストレージ アカウントの **[アクセス制御 (IAM)]** ウィンドウで、 **[+ 追加]** を選択し、ドロップダウン メニューで **[ロール割り当ての追加]** を選択します。 
1. **[ロール割り当ての追加]** ウィンドウで、次のように設定し、 **[保存]** を選択します。

   |設定|値|
   |---|---|
   |Role|**記憶域ファイル データの SMB 共有の管理者特権共同作成者**|
   |アクセスの割り当て先|**ユーザー、グループ、またはサービス プリンシパル**|
   |Select|**az140-wvd-admins**|

   > **注**:**az140-wvd-admins** グループには、共有権限を構成するために使用する **wvdadmin1** ユーザー アカウントが含まれています。 

1. 前の 2 つの手順を繰り返して、次の役割の割り当てを構成します。

   |設定|値|
   |---|---|
   |Role|**記憶域ファイル データの SMB 共有の管理者特権共同作成者**|
   |アクセスの割り当て先|**ユーザー、グループ、またはサービス プリンシパル**|
   |Select|**az140-hosts-42-p1**|

   |設定|値|
   |---|---|
   |Role|**ストレージ ファイル データの SMB 共有の閲覧者**|
   |アクセスの割り当て先|**ユーザー、グループ、またはサービス プリンシパル**|
   |Select|**az140-wvd-users**|

   > **注**:Azure Virtual Desktop のユーザーとホストには、少なくともファイル共有への読み取りアクセス権が必要です。

1. [ストレージ アカウント] ウィンドウの左側の垂直メニューにある **[データ ストレージ]** セクションで、 **[ファイル共有]** を選択し、 **[+ ファイル共有]** を選択します。
1. **[新しいファイル共有]** で、次の設定を指定し、 **[作成]** を選択します (他の設定は既定値のままにします)。

   |設定|[値]|
   |---|---|
   |名前|**az140-42-msixvhds**|

1. Azure portal を表示している Microsoft Edge のファイル共有の一覧で、新しく作成したファイル共有を選択します。 

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、**コマンド プロンプト** を起動し、 **[コマンド プロンプト]** ウィンドウから以下を実行して、ドライブを **az140-42-msixvhds** 共有にマップし (`<storage-account-name>` プレースホルダーをストレージ アカウントの名前に置き換えます)、コマンドが正常に完了することを確認します。

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-42-msixvhds
   ```

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、 **[コマンド プロンプト]** ウィンドウから次のコマンドを実行して、セッション ホストのコンピューター アカウントに必要な NTFS アクセス許可を付与します。

   ```cmd
   icacls Z:\ /grant ADATUM\az140-hosts-42-p1:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-users:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-admins:(OI)(CI)(F) /T
   ```

   >**注**:**ADATUM\\wvdadmin1** としてサインインしているときに、ファイル エクスプローラーを使用して、これらのアクセス許可を設定することもできます。 

   >**注**:次に、MSIX アプリのアタッチ機能を検証します

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** ウィンドウから以下を実行して、前の演習で作成した VHD ファイルを、この演習で先ほど作成した Azure Files 共有にコピーします。

   ```powershell
   New-Item -ItemType Directory -Path 'Z:\packages' 
   Copy-Item -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Destination 'Z:\packages' -Force
   ```

#### <a name="task-3-mount-and-register-the-msix-app-attach-image-on-azure-virtual-desktop-session-hosts"></a>タスク 3:MSIX アプリのアタッチ イメージを Azure Virtual Desktop セッションホストにマウントして登録する

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、Azure portal を表示している Microsoft Edge ウィンドウで「**Azure Virtual Desktop**」を検索して選択し、 **[Azure Virtual Desktop]** ウィンドウの左側の垂直メニューにある **[管理]** セクションで、 **[ホスト プール]** を選択します。
1. **[Azure Virtual Desktop \| ホスト プール]** ウィンドウで、ホスト プールの一覧から **[az140-21-hp1]** エントリを選択します。
1. **[az140-21-hp1 \| プロパティ]** ウィンドウの左側の垂直メニューにある **[管理]** セクションで、 **[MSIX パッケージ]** を選択します。
1. **[az140-21-hp1 \| MSIX パッケージ]** ウィンドウで、 **[+ 追加]** をクリックします。
1. **[MSIX パッケージの追加]** ウィンドウの **[MSIX イメージのパス]** テキストボックスに、**XmlNotepad.vhd** ファイルへのパスを `\\<storage-account-name>.file.core.windows.net\az140-42-msixvhds\packages\XmlNotepad.vhd`の形式で入力し (`<storage-account-name>` プレースホルダーを、**az140-42-msixvhds** ファイル共有をホストするストレージ アカウントの名前に置き換えます)、 **[追加]** をクリックします。
1. **[MSIX パッケージの追加]** ブレードで、次のように設定してから、 **[追加]** をクリックします。

   |設定|[値]|
   |---|---|
   |MSIX イメージのパス|**\\<storage-account-name>.file.core.windows.net\az140-42-msixvhds\XmlNotepad.vhd**。ここで、プレースホルダー `<storage-account-name>` は、**az140-42-msixvhds** ファイル共有をホストするストレージ アカウントの名前を指定します|
   |MSIX パッケージ|パッケージ作成中に生成される名前|
   |Display name|**XML Notepad**|
   |登録の種類|**オンデマンド**|
   |State|**Active**|

#### <a name="task-4-publish-msix-apps-to-an-application-group"></a>タスク 4:MSIX アプリをアプリケーション グループに公開する

> **注**:MSIX アプリをリモート アプリとデスクトップ アプリ グループの両方に公開します。 

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、Azure portal を表示している Microsoft Edge ウィンドウで、「**Azure Virtual Desktop**」を検索して選択し、 **[Azure Virtual Desktop]** ウィンドウの左側の垂直メニューにある **[管理]** セクションで、 **[アプリケーション グループ]** を選択します。
1. **[Azure Virtual Desktop \| アプリケーション グループ]** ウィンドウで、 **[az140-21-hp1-Utilities-RAG]** アプリケーション グループ エントリを選択します。
1. **[az140-21-hp1-Utilities-RAG]** ブレードの左側にある垂直メニューの **[管理]** セクションで、 **[アプリケーション]** を選択します。 
1. **[az140-21-hp1-Utilities-RAG | アプリケーション]** ウィンドウで、 **[+ 追加]** をクリックします。
1. **[アプリケーションの追加]** ウィンドウで、次のように設定し、 **[保存]** を選択します。

   |設定|[値]|
   |---|---|
   |アプリケーション ソース|**MSIX パッケージ**|
   |MSIX パッケージ|イメージに含まれるパッケージを表す名前|
   |MSIX アプリケーション|**XMLNOTEPAD**|
   |アプリケーション名|**XML Notepad**|
   |Display name|**XML Notepad**|
   |説明|**XML Notepad**|
   |アイコン パス|**C:\Program Files\WindowsApps\XmlNotepad_2.8.0.0_x64__4vm7ty4fw38e8\VFS\ProgramFilesX86\LovettSoftware\XmlNotepad\XmlNotepad.exe**|
   |Icon Index (アイコン インデックス)|**0**|

1. **[Azure Virtual Desktop \| アプリケーション グループ]** ウィンドウに戻り、 **[az140-21-hp1-DAG]** アプリケーション グループ エントリを選択します。
1. **[az140-21-hp1-DAG]** ブレードの左側にある垂直メニューの **[管理]** セクションで、 **[アプリケーション]** を選択します。 
1. **[az140-21-hp1-DAG | アプリケーション]** ウィンドウで、 **[+ 追加]** をクリックします。
1. **[アプリケーションの追加]** ウィンドウで、次のように設定し、 **[保存]** を選択します。

   |設定|[値]|
   |---|---|
   |アプリケーション ソース|**MSIX パッケージ**|
   |MSIX パッケージ|イメージに含まれるパッケージを表す名前|
   |アプリケーション名|**XML Notepad**|
   |Display name|**XML Notepad**|
   |説明|**XML Notepad**|

#### <a name="task-5-validate-the-functionality-of-msix-app-attach"></a>タスク 5:MSIX アプリのアタッチ機能を検証する

1. **az140-cl-vm42** へのリモート デスクトップ セッション内で、Microsoft Edge を起動し、[Windows デスクトップ クライアントのダウンロード ページ](https://go.microsoft.com/fwlink/?linkid=2068602)に移動し、ダウンロードが完了したら、 **[ファイルを開く]** を選択してインストールを開始します。 **[リモート デスクトップ セットアップ]** ウィザードの **[インストールのスコープ]** ページで、 **[このコンピューターのすべてのユーザー用にインストール]** オプションを選択し、 **[インストール]** をクリックします。 
1. インストールが完了したら、 **[セットアップの終了時にリモート デスクトップを起動する]** チェックボックスがオンになっていることを確認し、 **[完了]** をクリックしてリモート デスクトップ クライアントを起動します。
1. **[リモート デスクトップ]** クライアント ウィンドウで、 **[サブスクライブする]** を選択し、プロンプトが表示されたら、ユーザー プリンシパル名 **aduser1** と、このユーザー アカウントの作成時に設定したパスワードを使用してサインインします。 
1. プロンプトが表示されたら、 **[すべてのアプリにサインインしたままにする]** ウィンドウで、 **[組織にデバイスの管理を許可する]** チェックボックスをオフにし、 **[いいえ、このアプリのみにサインインします]** をクリックします。
1. **リモート デスクトップ** クライアント ウィンドウの **[az140-21-ws1]** セクションで、 **[XML Notepad]** アイコンをダブルクリックし、プロンプトが表示されたら、パスワードを入力して、XML Notepad が正常に起動することを確認します。


### <a name="exercise-4-stop-and-deallocate-azure-vms-provisioned-and-used-in-the-lab"></a>演習 4:ラボでプロビジョニングおよび使用された Azure VM を停止および割り当て解除する

この演習の主なタスクは次のとおりです。

1. ラボでプロビジョニングおよび使用された Azure VM を停止および割り当て解除する

>**注**:この演習では、このラボでプロビジョニングおよび使用した Azure VM を割り当て解除し、対応するコンピューティング料金を最小化します

#### <a name="task-1-deallocate-azure-vms-provisioned-and-used-in-the-lab"></a>タスク 1:ラボでプロビジョニングおよび使用された Azure VM を割り当て解除する

1. ラボ コンピューターに切り替え、Azure portal を表示している Web ブラウザーの画面で、**Cloud Shell** ペイン内の **PowerShell** シェル セッションを開きます。
1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成および使用されたすべての Azure VM を一覧表示します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   Get-AzVM -ResourceGroup 'az140-42-RG'
   ```

1. Cloud Shell ペインの PowerShell セッションから、以下を実行して、このラボで作成および使用されたすべての Azure VM を停止して割り当てを解除します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   Get-AzVM -ResourceGroup 'az140-42-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注**:コマンドは非同期で実行されるので (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に停止されて割り当てが解除されるまでに数分かかります。
