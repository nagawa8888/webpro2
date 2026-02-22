graph TD
    %% ----- Local Environment -----
    subgraph Local Environment [ユーザー端末 / 開発環境 (macOS)]
        
        subgraph PythonServer [3. yt-python-audio-detection\n(Python)]
            Mic[マイク] --> AudioDetector[指パッチン検知\n(FFT & TFLite)]
            Camera[Webカメラ] --> HandDetector[ジェスチャー検知\n(OpenCV & MediaPipe)]
            AudioDetector --> WSServer[WebSocket サーバー\nws://localhost:8765]
            HandDetector --> WSServer
        end

        subgraph MacApp [2. yt-mac-menu\n(Swift / SwiftUI)]
            UI[メニューバー UI / 設定]
            InputMonitor[ショートカット検知]
            WSClient[WebSocket クライアント]
            GitExecutor[ローカルGit操作\n(差分・ブランチ情報取得)]
            APIClient[AWS連携クライアント]

            WSServer <-->|コマンド送信・イベント受信| WSClient
            WSClient -->|アクション発火| GitExecutor
            InputMonitor -->|ショートカット発火| GitExecutor
            GitExecutor <-->|git status / git diff| LocalGit[(ローカル Git リポジトリ)]
            GitExecutor --> APIClient
        end
    end

    %% ----- AWS Cloud -----
    subgraph AWS Cloud [1. yt-aws-backend\n(Node.js / Hono on AWS)]
        APIGateway[AWS API Gateway]
        Lambda[AWS Lambda]
        Bedrock[AWS Bedrock\n(Claude 3 Haiku)]
        
        APIClient -->|HTTP POST /github/commit_push\n(ファイル差分, メタデータ)| APIGateway
        APIGateway --> Lambda
        Lambda <-->|差分を渡し、コミットメッセージと\nPRタイトル・本文の生成を依頼| Bedrock
    end

    %% ----- External Services -----
    subgraph External Services [GitHub]
        GitHubAPI[(GitHub API)]
        Lambda <-->|1. ブランチ作成\n2. 変更のコミット\n3. Pull Request作成| GitHubAPI
    end
