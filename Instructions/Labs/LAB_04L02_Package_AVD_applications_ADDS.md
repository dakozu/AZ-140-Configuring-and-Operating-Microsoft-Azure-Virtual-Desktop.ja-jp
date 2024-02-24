---
lab:
  title: 'ラボ: Azure Virtual Desktop アプリケーション (AD DS) のパッケージ化'
  module: 'Module 4: Manage User Environments and Apps'
---

# ラボ - Azure Virtual Desktop アプリケーション (AD DS) のパッケージ化
# 受講生用ラボ マニュアル

## ラボの依存関係

- Azure サブスクリプション
- Azure サブスクリプションに関連付けられた Microsoft Entra テナントの全体管理者ロールと、Azure サブスクリプションの所有者または共同作成者ロールを持つ Microsoft アカウントまたは Microsoft Entra アカウント
- 完了したラボ: **Azure Virtual Desktop (AD DS) のデプロイを準備する**
- 完了したラボ: **Azure Virtual Desktop プロファイル管理 (AD DS)**
- 実施するラボ - **WVD (AD DS) に対する条件付きアクセス ポリシーを構成する**

## 推定所要時間

60 分

## ラボのシナリオ

Active Directory Domain Services (AD DS) 環境で Azure Virtual Desktop アプリケーションをパッケージ化してデプロイする必要があります。

## 目標
  
このラボを完了すると、次のことができるようになります。

- MSIX アプリ パッケージを準備して作成する
- Microsoft Entra DS 環境で Azure Virtual Desktop の MSIX アプリのアタッチ イメージを実装する
- AD DS 環境で Azure Virtual Desktop の MSIX アプリ アタッチを実装する

## ラボ ファイル

-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json
-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json

## 手順

>**重要**: Microsoft は、**Azure Active Directory** (**Azure AD**) の名称を **Microsoft Entra ID** に変更しました。 この変更の詳細については、「[Azure Active Directory の新しい名前](https://learn.microsoft.com/en-us/entra/fundamentals/new-name)」を参照してください。 これは現在進行中の取り組みであるため、個々の演習を進めるときに、ラボの指示とインターフェイスの要素間にまだ不一致が残っている可能性があります。 この点を念頭に置いてください (特に、このラボでは、**Microsoft Entra Connect** は **Azure Active Directory Connect** の新しい名称です)。

### 演習 1: MSIX アプリ パッケージの準備と作成

この演習の主なタスクは次のとおりです。

1. Azure Virtual Desktop セッション ホストの構成を準備する
1. Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする
1. MSIX パッケージ用に Windows 10 を実行している Azure VM を準備する
1. 署名証明書を生成する
1. パッケージにソフトウェアをダウンロードする
1. MSIX パッケージ作成ツールをインストールする
1. MSIX パッケージを作成する

#### タスク 1: Azure Virtual Desktop セッション ホストの構成を準備する

1. ラボのコンピューターから Web ブラウザーを起動し、[Azure portal](https://portal.azure.com) に移動して、このラボで使用しているサブスクリプションで所有者のロールがあるユーザー アカウントの資格情報を指定してサインインします。
1. ラボのコンピューターで、Azure portal を表示している Web ブラウザー ウィンドウで、**[Cloud Shell]** ペイン内の **PowerShell** シェル セッションを開きます。
1. [Cloud Shell] ペインの [PowerShell] セッションから、次を実行して、このラボで使用する Azure VM をホストする Azure Virtual Desktop セッションを開始します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**注**: このコマンドは非同期で実行されるため (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に開始されるまでには数分かかります。 

   >**注**: 前のラボの最初のタスク (AVD プロファイルの実装と管理) において az140-21-RG リソース グループのセッション ホストで PSRemoting を有効にした場合は、Azure VM の起動を待たずに次のタスクに直接進むことができます。 az140-21-RG リソース グループのセッション ホストで PSRemoting を以前に有効にしていない場合は、VM が起動するのを待ってから、次のコマンドを実行します。

1. **Cloud Shell** の PowerShell セッションから、次を実行して、セッション ホストで PowerShell リモート処理を有効にします。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```
   
#### タスク 2: Azure Resource Manager クイックスタート テンプレートを使用して Windows 10 を実行する Azure VM をデプロイする

1. ラボ コンピューターから、Azure portal を表示している Web ブラウザー ウィンドウの Cloud Shell ペインのツールバーで、**[ファイルのアップロード/ダウンロード]** アイコンを選択し、ドロップダウン メニューで **[アップロード]** を選択し、**\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json** ファイルと **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json** ファイルを Cloud Shell ホーム ディレクトリにアップロードします。
1. [Cloud Shell] ペインの [PowerShell] セッションから、次を実行して、MSIX パッケージの作成に使用する Windows 10 を実行する Azure VM をデプロイし、Microsoft Entra DS ドメインに参加します。

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

#### タスク 3: MSIX パッケージ化のために Windows 10 を実行している Azure VM を準備する

1. ラボ コンピューターの Azure portal で、「**仮想マシン**」を検索して選択し、**[仮想マシン]** ウィンドウの仮想マシンの一覧で **[az140-cl-vm42]** エントリを選択します。 これにより、**[az140-cl-vm42]** ウィンドウが開きます。
1. **[az140-cl-vm42]** ウィンドウで **[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、**[az140-cl-vm42 \| 接続]** ウィンドウの **[Bastion]** タブで **[Bastion を使用する]** を選択します。
1. プロンプトが表示されたら、**wvdadmin1@adatum.com** のユーザー名と、このユーザー アカウントを作成したときに設定したパスワードを使用してサインインします。 
1. **az140-cl-vm42** への Bastion セッション内で、**Windows PowerShell ISE** を管理者として起動します。**[Administrator: Windows PowerShell ISE]** コンソールから、次を実行して、MSIX パッケージ化のオペレーティング システムを準備します。

   ```powershell
   Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
   reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
   reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
   reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
   ```

   > **注**: これらのレジストリの最後の変更は、ユーザー アクセス制御を無効にします。 これは技術的には必要ありませんが、このラボに示されているプロセスが簡略化されます。

#### タスク 4: 署名証明書を生成する

> **注**: この実習ラボでは、自己署名証明書を使用します。 運用環境では、使用目的に応じて、パブリック証明機関または内部証明機関によって発行された証明書を使用する必要があります。

1. **az140-cl-vm42** への Bastion セッション内で、**[Administrator: Windows PowerShell ISE]** コンソールから、以下を実行して、共通名属性を **Adatum** に設定した自己署名証明書を生成し、**ローカル マシン**証明書ストアの**個人用**フォルダーに証明書を保存します。

   ```powershell
   New-SelfSignedCertificate -Type Custom -Subject "CN=Adatum" -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -CertStoreLocation "cert:\LocalMachine\My"
   ```

1. **[Administrator: Windows PowerShell ISE]** コンソールから以下を実行して、ローカル マシン証明書ストアを対象とする **[証明書]** コンソールを起動します。

   ```powershell
   certlm.msc
   ```

1. **[証明書]** コンソール ペインで、**[個人用]** フォルダーを展開し、**[証明書]** サブフォルダーを選択し、**Adatum** 証明書を右クリックし、右クリック メニューで **[すべてのタスク]** の後に **[エクスポート]** を選択します。 これで、**証明書のエクスポート ウィザード**が起動します。 
1. **証明書のエクスポート ウィザード**の **「証明書のエクスポート ウィザードへようこそ」** ページで、**[次へ]** を選択します。
1. **証明書のエクスポート ウィザード**の **「秘密キーをエクスポートする」** ページで、**[はい、秘密キーをエクスポートします]** オプションを選択し、**[次へ]** を選択します。
1. **証明書のエクスポート ウィザード**の **「ファイル形式のエクスポート」** ページで、**[すべての拡張プロパティをエクスポートする]** チェックボックスを選択し、**[証明書のプライバシーを有効にする]** チェックボックスをオフにして、**[次へ]** を選択します。
1. **[証明書エクスポート]** ウィザードの **[セキュリティ]** ページで、**[パスワード]** チェックボックスをオンにし、下のテキストボックスに「**Pa55w.rd1234**」と入力して、**[次へ]** を選択します。
1. **[証明書エクスポート]** ウィザードの **[エクスポートするファイル]** ページの **[ファイル名]** テキストボックスで **[参照]** を選択し、**[名前を付けて保存]** ダイアログ ボックスで **C:\\Allfiles\\Labs\\04** フォルダーに移動し (最初にフォルダーを作成します)、**[ファイル名]** テキストボックスに「**adatum.pfx**」と入力し、**[保存]** を選択します。
1. **[証明書エクスポート]** ウィザードの **[エクスポートするファイル]** ページに戻り、テキストボックスにエントリ "**C:\\Allfiles\\Labs\\04\\adatum.pfx**" が含まれていることを確認して、**[次へ]** を選択します。
1. **証明書のエクスポート ウィザード**の **「証明書のエクスポート ウィザードの完了」** ページで、**[完了]** を選択し、**[OK]** を選択してエクスポートが成功したことを確認します。 

   > **注**: 自己署名証明書を使用しているため、ターゲット セッション ホストの**信頼されたユーザー**証明書ストアにインストールする必要があります。

1. **[Administrator: Windows PowerShell ISE]** コンソールから以下を実行して、新しく生成された証明書をターゲット セッション ホストの**信頼できるユーザー**証明書ストアにインストールします。

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

#### タスク 5: パッケージにソフトウェアをダウンロードする

1. **az140-cl-vm42** への Bastion セッション内で、**Microsoft Edge** を起動し、**https://github.com/microsoft/XmlNotepad** を参照します。
1. **[microsoft/XmlNotepad** **readme.md]** ページで、ダウンロード可能なスタンドアロン インストーラーのダウンロード リンクを選択し、圧縮されたインストール ファイルをダウンロードします。
1. **az140-cl-vm42** への Bastion セッション内で、ファイル エクスプローラーを起動し、**ダウンロード** フォルダーに移動し、圧縮ファイルを開き、圧縮ファイル内のフォルダー内のコンテンツをコピーして、**C:\\AllFiles\\Labs\\04\\** ディレクトリに貼り付けます。 

#### タスク 6: MSIX パッケージ ツールをインストールする

1. **az140-cl-vm42** への Bastion セッション内で、**Microsoft Store** アプリを起動します。
1. **Microsoft Store** アプリで、**「MSIX Packaging Tool」** を検索して選択し、**「MSIX Packaging Tool」** ページで **[取得]** を選択します。
1. プロンプトが表示されたら、サインインをスキップし、インストールが完了するのを待って **[開く]** を選択し、**[診断データの送信]** ダイアログ ボックスで **[拒否]** を選択します。 

#### タスク 7: MSIX パッケージを作成する

1. **az140-cl-vm42** への Bastion セッション内で、**[Administrator: Windows PowerShell ISE]** ウィンドウに切り替え、**[Administrator: Windows PowerShell ISE]** スクリプト ペインから次を実行して Windows Search サービスを無効にします。

   ```powershell
   $serviceName = 'wsearch'
   Set-Service -Name $serviceName -StartupType Disabled
   Stop-Service -Name $serviceName
   ```

1. **az140-cl-vm42** への Bastion セッション内で、**Administrator: Windows PowerShell ISE** コンソールから次を実行して、MSIX パッケージをホストするフォルダーを作成します。

   ```powershell
   New-Item -ItemType Directory -Path 'C:\AllFiles\Labs\04\XmlNotepad' -Force
   ```

1. **az140-cl-vm42** への Bastion セッション内で、**Administrator: Windows PowerShell ISE** コンソールから以下を実行して、抽出されたインストーラー ファイルから Zone.Identifier 代替データ ストリームを削除します。これには、インターネットからダウンロードされたことを示す値 "3" が含まれています。

   ```powershell
   Get-ChildItem -Path 'C:\AllFiles\Labs\04' -Recurse -File | Unblock-File
   ```

1. **az140-cl-vm42** への Bastion セッション内で、**MSIX Packaging Tool** インターフェイスに切り替え、**「タスクの選択」** ページで **[アプリケーション パッケージ - アプリ パッケージ エントリの作成]** を選択します。 これにより、**[新しいパッケージの作成]** ウィザードが開始されます。
1. **[新しいパッケージの作成]** ウィザードの **[環境の選択]** ページで、**[このコンピューターにパッケージを作成する]** オプションが選択されていることを確認し、**[次へ]** を選択して、**MSIX パッケージ作成ツールドライバー**のインストールを待ちます。
1. **[新しいパッケージの作成]** ウィザードの **「コンピューターの準備」** ページで、推奨事項を確認します。 再起動が保留中の場合は、オペレーティング システムを再起動し、**wvdadmin1@adatum.com** アカウントを使用してサインインし直し、続行する前に **MSIX パッケージ ツール**を再起動します。 

   >**注**: MSIX Packaging Tool では、Windows Update と Windows Search が一時的に無効になります。 この場合、Windows Search サービスは既に無効になっています。 

1. **[新しいパッケージの作成]** ウィザードの **「コンピューターの準備」** ページで、**[次へ]** をクリックします。
1. **[新しいパッケージの作成]** ウィザードの **[インストーラーの選択]** ページで、**[パッケージ化するインストーラーの選択]** テキスト ボックスの横にある **[参照]** を選択し、**[開く]** ダイアログ ボックスで **C:\\AllFiles\\Labs\\04** フォルダーを参照し、**XmlNotepadSetup.msi** を選択して、**[開く]** をクリックします。 
1. **[新しいパッケージの作成]** ウィザードの **[インストーラーの選択]** ページの **[署名の設定]** ドロップダウン リストで、**[証明書 (.pfx) を使用して署名する]** エントリを選択し、**[証明書の参照]** テキストボックスの横にある **[参照]** を選択して、**[開く]** ダイアログ ボックスで、**C:\\AllFiles\\Labs\\04** フォルダーに移動し、**adatum.pfx** ファイルを選択して、**[開く]** をクリックし、**[パスワード]** テキスト ボックスに 「**Pa55w.rd1234**」と入力して、**[次へ]** を選択します。
1. **[新しいパッケージの作成]** ウィザードの **「パッケージ情報」** ページで、パッケージ情報を確認し、発行元名が **CN=Adatum** に設定されていることを確認して、**[次へ]** を選択します。 これにより、ダウンロードしたソフトウェアのインストールがトリガーされます。
1. **[XML メモ帳セットアップ]** ウィンドウで、使用許諾契約書の条項に同意し、**[インストール]** を選択し、インストールが完了したら、**[XML メモ帳の起動]** チェックボックスを選択し、**[完了]** を選択します。
1. メッセージが表示されたら、**[XML メモ帳 Analytics]** ウィンドウで **[いいえ]** を選択し、XML メモ帳が実行されていることを確認し、閉じて、**[MSIX パッケージ ツール]** ウィンドウで **[新しいパッケージの作成]** ウィザードに戻り、**[次へ]** を選択します。

   > **注**: この場合、インストールを完了するために再起動は必要ありません。

1. **[新しいパッケージの作成]** ウィザードの **「最初の起動タスク」** ページで、指定された情報を確認し、**[次へ]** を選択します。
1. "**完了しましたか?**" というプロンプトが表示されたら、**[はい、次に進む]** を選択します。
1. **[新しいパッケージの作成]** ウィザードの **「サービス レポート」** ページで、サービスが一覧に表示されていないことを確認し、**[次へ]** を選択します。
1. **[新しいパッケージの作成]** ウィザードの **[パッケージの作成]** ページの **[保存場所]** テキストボックスに、「**C:\\Allfiles\\Labs\\04\\XmlNotepad\XmlNotepad.msix**」と入力し、**[作成]** をクリックします。
1. **[パッケージが正常に作成されました]** ダイアログ ボックスで、保存したパッケージの場所を書き留め、**[閉じる]** を選択します。
1. ファイル エクスプローラー ウィンドウに切り替え、**C:\\Allfiles\\Labs\\04\\XmlNotepad** フォルダーに移動し、*.msix ファイルと *.xml ファイルが含まれていることを確認します。
1. **XmlNotepad.msix** ファイルを **C:\\Allfiles\\Labs\\04** フォルダーにコピーします。


### 演習 2: Microsoft Entra DS 環境で Azure Virtual Desktop 用の MSIX アプリ アタッチ イメージを実装する

この演習の主なタスクは次のとおりです。

1. Window 10 Enterprise Edition を実行している Azure VM で Hyper-V を有効にする
1. MSIX アプリ アタッチ イメージを作成する

#### タスク 1: Window 10 Enterprise Edition を実行している Azure VM で Hyper-V を有効にする

1. **az140-cl-vm42** への Bastion セッション内で、**Administrator: Windows PowerShell ISE** コンソールから、以下を実行して、MSIX アプリ アタッチ用のターゲット Azure Virtual Desktop ホストを準備します。 

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

1. **az140-cl-vm42** への Bastion セッション内で、**Administrator: Windows PowerShell ISE** コンソールで以下を実行し、Azure Virtual Desktop ホストに Hyper-V PowerShell モジュールを含む Hyper-V とその管理ツールをインストールします。

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
      }
   }
   ```

1. ターゲット オペレーティング システムを起動するかどうかを尋ねられたら、**[はい]** を選択します。
1. **az140-cl-vm42** への Bastion セッション内で、**Administrator: Windows PowerShell ISE** コンソールで以下を実行し、Hyper-V PowerShell モジュールを含む Hyper-V とその管理ツールをローカル コンピューターにインストールします。

   ```powershell
   Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
   ```

1. Hyper-V コンポーネントのインストールが完了したら、**[はい]** を選択してオペレーティング システムを再起動します。 再起動後、このユーザー アカウントの作成時に設定した **wvdadmin1@adatum.com** のユーザー名とパスワードでサインインし直します。

#### タスク 2: MSIX アプリ アタッチ イメージを作成する

1. **az140-cl-vm42** への Bastion セッション内で、**Microsoft Edge** を起動し、**https://aka.ms/msixmgr** を参照します。 これにより、**msixmgr.zip** ファイル (MSIX mgr ツール アーカイブ) が "**ダウンロード**" フォルダーに自動的にダウンロードされます。
1. ファイル エクスプローラーで、"**ダウンロード**" フォルダーに移動し、圧縮ファイルを開いて、**x64** フォルダーの内容を **C:\\AllFiles\\Labs\\04** フォルダーにコピーします。 
1. **az140-cl-vm42** への Bastion セッション内で、**Windows PowerShell ISE** を管理者として起動し、**[Administrator: Windows PowerShell ISE]** スクリプト ペインから次を実行して、MSIX アプリ アタッチ イメージとして機能する VHD ファイルを作成します。

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Allfiles\Labs\04\MSIXVhds' -Force
   New-VHD -SizeBytes 128MB -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Dynamic -Confirm:$false
   ```

1. **[Administrator: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、新しく作成された VHD ファイルをマウントします。

   ```powershell
   $vhdObject = Mount-VHD -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Passthru
   ```

1. **[Administrator: Windows PowerShell ISE]** スクリプト ペインから以下を実行してディスクを初期化し、新しいパーティションを作成してフォーマットし、使用可能な最初のドライブ文字を割り当てます。

   ```powershell
   $disk = Initialize-Disk -Passthru -Number $vhdObject.Number
   $partition = New-Partition -AssignDriveLetter -UseMaximumSize -DiskNumber $disk.Number
   Format-Volume -FileSystem NTFS -Confirm:$false -DriveLetter $partition.DriveLetter -Force
   ```

   > **注**: F: ドライブのフォーマットを求めるポップアップ ウィンドウが表示されたら、**[キャンセル]** を選択します。

1. **[Administrator: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、MSIX ファイルをホストするフォルダー構造を作成し、前のタスクで作成した MSIX パッケージに解凍します。

   ```powershell
   $appName = 'XmlNotepad'
   New-Item -ItemType Directory -Path "$($partition.DriveLetter):\Apps" -Force
   Set-Location -Path 'C:\AllFiles\Labs\04\x64'
   .\msixmgr.exe -Unpack -packagePath ..\$appName.msix -destination "$($partition.DriveLetter):\Apps" -applyacls
   ```

1. **az140-cl-vm42** への Bastion セッション内のファイル エクスプローラーで、**F:\\Apps** フォルダーに移動し、その内容を確認します。 このフォルダーへのアクセスを取得するように求められたら、**[続行]** を選択します。
1. **az140-cl-vm42** への Bastion セッション内で、**[Administrator: Windows PowerShell ISE]** コンソールから、以下を実行して、MSIX イメージとして機能する VHD ファイルをマウント解除します。

   ```powershell
   Dismount-VHD -Path "C:\Allfiles\Labs\04\MSIXVhds\$appName.vhd" -Confirm:$false
   ```

### 演習 3: Azure Virtual Desktop セッション ホストに MSIX アプリ アタッチを実装する

この演習の主なタスクは次のとおりです。

1. Azure Virtual Desktop ホストを含む Active Directory グループを構成する
1. MSIX アプリ アタッチ用に Azure Files 共有を設定する
1. MSIX アプリ アタッチ イメージを Azure Virtual Desktop セッション ホストにマウントして登録する
1. MSIX アプリをアプリケーション グループに公開する
1. MSIX アプリ アタッチの機能を検証する

#### タスク 1: Azure Virtual Desktop ホストを含む Active Directory グループを構成する

1. ラボ コンピューターに切り替え、Azure portal を表示する Web ブラウザーで、「**仮想マシン**」を検索して選択し、**[仮想マシン]** ウィンドウから **[az140-dc-vm11]** を選択します。
1. **[az140-dc-vm11]** ウィンドウで **[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択し、**[az140-dc-vm11 \| 接続]** ウィンドウの **[Bastion]** タブで **[Bastion を使用する]** を選択します。
1. プロンプトが表示されたら、次の資格情報を入力し、**[接続]** を選択します。

   |設定|値|
   |---|---|
   |[ユーザー名]|**Student**|
   |パスワード|**Pa55w.rd1234**|

1. **az140-dc-vm11** への Bastion セッション内で、**Windows PowerShell ISE** を管理者として起動します。
1. **[Administrator: Windows PowerShell ISE]** スクリプト ペインから、以下を実行して、このラボで使用する Microsoft Entra テナントに同期される AD DS グループ オブジェクトを作成します。

   ```powershell
   $ouPath = "OU=WVDInfra,DC=adatum,DC=com"
   New-ADGroup -Name 'az140-hosts-42-p1' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

   > **注**: このグループを使用して、Azure Virtual Desktop ホストに **az140-42-msixvhds** ファイル共有へのアクセス許可を付与します。

1. **[管理者: Windows PowerShell ISE]** コンソールから、以下を実行して、前の手順で作成したグループにメンバーを追加します。

   ```powershell
   Get-ADGroup -Identity 'az140-hosts-42-p1' | Add-AdGroupMember -Members 'az140-21-p1-0$','az140-21-p1-1$','az140-21-p1-2$'
   ```

1. **[Administrator: Windows PowerShell ISE]** スクリプト ペインから以下を実行して、'az140-hosts-42-p1' グループのメンバーであるサーバーを再起動します。

   ```powershell
   $hosts = (Get-ADGroup -Identity 'az140-hosts-42-p1' | Get-ADGroupMember | Select-Object Name).Name
   $hosts | ForEach-Object {Restart-Computer -ComputerName $_ -Force}
   ```

   > **注**: この手順により、グループ メンバーシップの変更が有効になります。 

1. **az140-dc-vm11** への Bastion セッション内の **[スタート]** メニューで、**Microsoft Entra Connect** フォルダーを展開し、**Microsoft Entra Connect** を選択します。
1. **[Microsoft Entra Connect]** ウィンドウの **「Microsoft Entra Connect へようこそ」** ページで、**[構成]** を選択します。
1. **[Microsoft Entra Connect]** ウィンドウの **「追加のタスク」** ページで、**[同期オプションをカスタマイズする]** を選択し、**[次へ]** を選択します。
1. **[Microsoft Entra Connect]** ウィンドウの **「Microsoft Entra に接続」** ページで、このタスクで先ほど特定した **aadsyncuser** ユーザー アカウントのユーザー プリンシパル名と、このユーザー アカウントの作成時に設定したパスワードを使用して認証します。
1. **[Microsoft Entra Connect]** ウィンドウの **「ディレクトリの接続」** ページで、**[次へ** を選択します。
1. **[Microsoft Entra Connect]** ウィンドウの **「ドメインと OU のフィルタリング」** ページで、**[選択したドメインと OU を同期する]** オプションが選択されていることを確認し、**adatum.com** ノードを展開し、**WVDInfra** OU の横にあるチェックボックスを選択して (他の選択されたチェックボックスは変更しないままにしておきます)、**[次へ]** を選択します。
1. **[Microsoft Entra Connect]** ウィンドウの **「オプション機能」** ページで、既定の設定をそのまま使用して、**[次へ]** を選択します。
1. **[Microsoft Entra Connect]** ウィンドウの **「構成の準備完了」** ページで、**[構成が完了したら、同期プロセスを開始してください]** チェックボックスが選択されていることを確認し、**[構成]** を選択します。
1. **「構成が完了しました」** ページの情報を確認し、**[終了]** を選択して、**[Mcrosoft Entra Connect]** ウィンドウを閉じます。
1. **az140-dc-vm11** への Bastion セッション内で、Microsoft Edge を起動して、[Azure portal](https://portal.azure.com) に移動します。 メッセージが表示されたら、このラボで使用している Azure サブスクリプションに関連付けられている Microsoft Entra テナントのグローバル管理者ロールを持つユーザー アカウントの Microsoft Entra 資格情報を使用してサインインします。
1. **az140-dc-vm11** への Bastion セッション内において、Azure portal が表示されている [Microsoft Edge] ウィンドウで、**「Azure Active Directory」** を検索して選択し、このラボに使用している Azure サブスクリプションに関連付けられている Microsoft Entra テナントに移動します。
1. [Azure Active Directory] ブレードの左側にある垂直方向のメニュー バーの **[管理]** セクションで、**[グループ]** をクリックします。 
1. **[グループ | すべてのグループ]** ウィンドウで、グループの一覧から **[az140-hosts-42-p1]** エントリを選択します。

   > **注**: グループを表示するには、ページの更新が必要になる場合があります。

1. **[az140-hosts-42-p1]** ブレードの左側にある垂直方向のメニューの **[管理]** セクションで、**[メンバー]** をクリックします。
1. **[az140-hosts-42-p1 | メンバー]** ウィンドウで、**[ダイレクト メンバー]** の一覧に、このタスクで先ほどグループに追加した Azure Virtual Desktop プールの 3 つのホストが含まれていることを確認します。

#### タスク 2: MSIX アプリ アタッチ用に Azure Files 共有を設定する

1. ラボのコンピューターで、Bastion セッション に戻って **az140-cl-vm42** に切り替えます。
1. **az140-cl-vm42** への Bastion セッションで Microsoft Edge を InPrivate モードで起動し、[Azure portal](https://portal.azure.com) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を指定してサインインします。

   > **注**: 必ず Microsoft Edge の InPrivate モードを使用してください。

1. **az140-cl-vm42** への Bastion セッション内において、Azure portal を表示する [Microsoft Edge] ウィンドウで **「ストレージ アカウント」** を検索して選択し、**[ストレージ アカウント]** ブレードで、ユーザー プロファイルをホストするように構成したストレージ アカウントを選択します。

   > **注**: ラボのこの部分は、ラボの **Azure Virtual Desktop プロファイル管理 (AD DS)** または **Azure Virtual Desktop プロファイル管理 (Microsoft Entra DS)** の完了に関連しています。

   > **注**: 運用環境のシナリオでは、別のストレージ アカウントの使用を検討する必要があります。 これには、ユーザー プロファイルをホストするストレージ アカウントに対して既に実装した Microsoft Entra DS 認証用にそのストレージ アカウントを構成する必要があります。 同じストレージ アカウントを使用して、個々のラボ間の重複する手順を最小限に抑えます。

1. [ストレージ アカウント] ウィンドウの左側の垂直メニューで、**[アクセス制御 (IAM)]** を選択します。
1. ストレージ アカウントの **[アクセス制御 (IAM)]** ブレードで、**[+ 追加]** を選択し、ドロップダウン メニューで **[ロール割り当ての追加]** を選択します。 
1. **[ロール割り当ての追加]** ウィンドウで、次のように設定し、**[保存]** を選択します。

   |設定|値|
   |---|---|
   |Role|**記憶域ファイル データの SMB 共有の管理者特権共同作成者**|
   |アクセスの割り当て先|**ユーザー、グループ、またはサービス プリンシパル**|
   |選択|**az140-wvd-admins**|

   > **注**: **az140-wvd-admins** グループには、共有アクセス許可の構成に使用する **wvdadmin1** ユーザー アカウントが含まれています。 

1. 前の 2 つの手順を繰り返して、次のロールの割り当てを構成します。

   |設定|値|
   |---|---|
   |Role|**記憶域ファイル データの SMB 共有の管理者特権共同作成者**|
   |アクセスの割り当て先|**ユーザー、グループ、またはサービス プリンシパル**|
   |選択|**az140-hosts-42-p1**|

   |設定|値|
   |---|---|
   |Role|**ストレージ ファイル データの SMB 共有の閲覧者**|
   |アクセスの割り当て先|**ユーザー、グループ、またはサービス プリンシパル**|
   |選択|**az140-wvd-users**|

   > **注**: Azure Virtual Desktop のユーザーとホストには、少なくともファイル共有への読み取りアクセス権が必要です。

1. ストレージ アカウント ブレードの左側の垂直メニューの **[データ ストレージ]** セクションで、**[ファイル共有]** を選択し、**[+ ファイル共有]** を選択します。
1. **[新しいファイル共有]** で、次の設定を指定し、**[作成]** を選択します (他の設定は既定値のままにします)。

   |設定|値|
   |---|---|
   |名前|**az140-42-msixvhds**|

1. Azure portal が表示されている Microsoft Edge のファイル共有の一覧で、新しく作成したファイル共有を選択します。 

1. **az140-cl-vm42** への Bastion セッション内で、**コマンド プロンプト**を起動し、**[コマンド プロンプト]** ウィンドウから以下を実行して、ドライブを **az140-42-msixvhds** 共有にマップし (`<storage-account-name>` プレースホルダーをストレージ アカウントの名前に置き換えます)、コマンドが正常に完了することを確認します。

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-42-msixvhds
   ```

1. **az140-cl-vm42** への Bastion セッション内で、**[コマンド プロンプト]** ウィンドウから次を実行して、セッション ホストのコンピューター アカウントに必要な NTFS アクセス許可を付与します。

   ```cmd
   icacls Z:\ /grant ADATUM\az140-hosts-42-p1:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-users:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-admins:(OI)(CI)(F) /T
   ```

   >**注**: **wvdadmin1\@adatum.com** としてサインインしているときに、ファイル エクスプローラーを使用して、これらのアクセス許可を設定することもできます。 

   >**注**: 次に、MSIX アプリ アタッチの機能を検証します。

1. **az140-cl-vm42** への Bastion セッション内で **[Administrator: Windows PowerShell ISE]** ウィンドウから以下を実行して、前の演習で作成した VHD ファイルを、この演習で先ほど作成した Azure Files 共有にコピーします。

   ```powershell
   New-Item -ItemType Directory -Path 'Z:\packages' 
   Copy-Item -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Destination 'Z:\packages' -Force
   ```

#### タスク 3: Azure Virtual Desktop セッション ホストに MSIX アプリ アタッチ イメージをマウントして登録する

1. **az140-cl-vm42** への Bastion セッション内において、Azure portal を表示している [Microsoft Edge] ウィンドウで **「Azure Virtual Desktop」** を検索して選択し、**[Azure Virtual Desktop]** ブレードの左側の垂直方向のメニューにある **[管理]** セクションで、**[ホスト プール]** を選択します。
1. **[Azure Virtual Desktop \| ホスト プール]** ウィンドウで、ホスト プールの一覧から **[az140-21-hp1]** エントリを選択します。
1. **[az140-21-hp1 \| プロパティ]** ウィンドウの左側の垂直メニューにある **[管理]** セクションで、**[MSIX パッケージ]** を選択します。
1. **[az140-21-hp1 \| MSIX パッケージ]** ウィンドウで、**[+ 追加]** をクリックします。
1. **[MSIX パッケージの追加]** ウィンドウの **[MSIX イメージのパス]** テキストボックスに、**XmlNotepad.vhd** ファイルへのパスを `\\<storage-account-name>.file.core.windows.net\az140-42-msixvhds\packages\XmlNotepad.vhd`の形式で入力し (`<storage-account-name>` プレースホルダーを、**az140-42-msixvhds** ファイル共有をホストするストレージ アカウントの名前に置き換えます)、**[追加]** をクリックします。
1. **[MSIX パッケージの追加]** ブレードで、次のように設定してから **[追加]** をクリックします。

   |設定|Value|
   |---|---|
   |MSIX イメージ パス|**\\\\\<storage-account-name\>.file.core.windows.net\\az140-42-msixvhds\\XmlNotepad.vhd** で、`<storage-account-name>` のプレースホルダーは、**az140-42-msixvhds** ファイル共有をホストするストレージ アカウントの名前を指定します。|
   |MSIX パッケージ|パッケージの作成時に生成される名前|
   |[表示名]|**XML Notepad**|
   |登録タイプ|**オンデマンド**|
   |都道府県|**アクティブ**|

#### タスク 4: MSIX アプリをアプリケーション グループに公開する

> **注**: MSIX アプリをリモート アプリとデスクトップ アプリ グループの両方に公開します。 

1. **az140-cl-vm42** への Bastion セッション内の Azure portal を表示している [Microsoft Edge] ウィンドウで、**「Azure Virtual Desktop」** を検索して選択し、**[Azure Virtual Desktop]** ブレードの左側の垂直方向のメニューにある **[管理]** セクションで、**[アプリケーション グループ]** を選択します。
1. **[Azure Virtual Desktop \| アプリケーション グループ]** ウィンドウで、**[az140-21-hp1-Utilities-RAG]** アプリケーション グループ エントリを選択します。
1. **[az140-21-hp1-Utilities-RAG]** ブレードの左側の垂直方向のメニューにある **[管理]** セクションで、**[アプリケーション]** を選択します。 
1. **[az140-21-hp1-Utilities-RAG \| アプリケーション]** ブレードで、**[+ 追加]** をクリックします。
1. **[アプリケーションの追加]** ブレードで **[基本]** タブと **[アイコン]** タブで次の設定を指定し、**[保存]** を選びます。

   |設定|Value|
   |---|---|
   |アプリケーション ソース|**アプリ アタッチ**|
   |Package|イメージに含まれるパッケージを表す名前|
   |アプリケーション|**XMLNOTEPAD**|
   |アプリケーション識別子|**XML Notepad**|
   |[表示名]|**XML Notepad**|
   |説明|**XML Notepad**|
   |アイコン ソース|**既定値**|

1. **[Azure Virtual Desktop \| アプリケーション グループ]** ウィンドウに戻り、**[az140-21-hp1-DAG]** アプリケーション グループ エントリを選択します。
1. **az140-21-hp1-DAG** ブレードの左側にある垂直方向のメニューの **[管理]** セクションで、**[アプリケーション]** を選択します。 
1. **[az140-21-hp1-DAG \| アプリケーション]** ブレードで、**[+ 追加]** をクリックします。
1. **[アプリケーションの追加]** ブレードで、次のように設定してから **[保存]** を選択します。

   |設定|Value|
   |---|---|
   |アプリケーション ソース|**MSIX パッケージ**|
   |MSIX パッケージ|イメージに含まれるパッケージを表す名前|
   |アプリケーション名|**XML Notepad**|
   |[表示名]|**XML Notepad**|
   |説明|**XML Notepad**|

#### タスク 5: MSIX アプリ アタッチの機能を検証する

1. **az140-cl-vm42** への Bastion セッション内で、Microsoft Edge を起動し、[「Windows デスクトップ クライアントのダウンロード」](https://go.microsoft.com/fwlink/?linkid=2068602)ページに移動し、ダウンロードが完了したら、**[ファイルを開く]** を選択してインストールを開始します。 **[リモート デスクトップ セットアップ]** ウィザードの **[インストールのスコープ]** ページで、**[このコンピューターのすべてのユーザー用にインストール]** オプションを選択し、**[インストール]** をクリックします。 
1. インストールが完了したら、**[セットアップの終了時にリモート デスクトップを起動する]** チェックボックスがオンになっていることを確認し、**[完了]** をクリックしてリモート デスクトップ クライアントを起動します。
1. **[リモート デスクトップ]** クライアント ウィンドウで、**[サブスクライブする]** を選択し、プロンプトが表示されたら、ユーザー プリンシパル名 **aduser1** と、このユーザー アカウントの作成時に設定したパスワードを使用してサインインします。 
1. プロンプトが表示されたら、**[すべてのアプリにサインインしたままにする]** ウィンドウで、**[組織にデバイスの管理を許可する]** チェックボックスをオフにして、**[いいえ、このアプリのみにサインインします]** をクリックします。
1. **[リモート デスクトップ]** クライアント ウィンドウの **[az140-21-ws1]** セクションで、プロンプトが表示されたら **XML メモ帳**アイコンをダブルクリックし、パスワードを入力して、XML メモ帳が正常に起動することを確認します。


### 演習 4: ラボでプロビジョニングおよび使用されている Azure VM を停止して割り当てを解除する

この演習の主なタスクは次のとおりです。

1. ラボでプロビジョニングおよび使用されている Azure VM を停止して割り当てを解除する

>**注**: この演習では、関連するコンピューティング料金を最小限に抑えるために、このラボでプロビジョニングおよび使用されている Azure VM の割り当てを解除します

#### タスク 1: ラボでプロビジョニングおよび使用されている Azure VM の割り当てを解除する

1. ラボのコンピューターに切り替え、Azure portal が表示されている Web ブラウザー ウィンドウにおいて、**[Cloud Shell]** ペイン内で **[PowerShell]** シェル セッションを開きます。
1. [Cloud Shell] ウィンドウの PowerShell セッションから、次を実行して、このラボで作成および使用されているすべての Azure VM を一覧表示します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   Get-AzVM -ResourceGroup 'az140-42-RG'
   ```

1. [Cloud Shell] ウィンドウの PowerShell セッションから、次を実行して、このラボで作成および使用したすべての Azure VM を停止して割り当てを解除します。

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   Get-AzVM -ResourceGroup 'az140-42-RG' | Stop-AzVM -NoWait -Force
   ```

   >**注**: このコマンドは非同期で実行されるため (-NoWait パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、Azure VM が実際に停止されて割り当てが解除されるまでに数分かかります。
