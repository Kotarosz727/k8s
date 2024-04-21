# Namespaceを作成

```
kubectl create ns taskapp
```

# Secretを作成

- secretの作成コマンド
    ```
    kubectl create secret generic test-secret --from-literal=password=gihyo_password
    ```

- genericはローカルファイルやkey-valueの値からsecretを作成するためのサブコマンド
- 作成したsecretを確認。Base64でエンコードされているはず
    ```
    kubectl get secret/test-secret -o yaml
    ```

- mysql接続情報のsecretを作成する  
- ファイル値を読み込み環境変数に設定する  
  ```
   MYSQL_ROOT_PASSWORD=$(cat ~/go/src/github.com/gihyodocker/taskapp/secrets/mysql_root_password)  

   MYSQL_USER_PASSWORD=$(cat ~/go/src/github.com/gihyodocker/taskapp/secrets/mysql_user_password)

   //check
   echo $MYSQL_ROOT_PASSWORD
   echo $MYSQL_USER_PASSWORD

   // 環境変数を元にsercret作成 (dry-run)
   kubectl create secret generic mysql --dry-run=client -o yaml \
   --from-literal=root_password=$MYSQL_ROOT_PASSWORD \
   --from-literal=user_password=$MYSQL_USER_PASSWORD > ~/k8s/taskapp/local/mysql-secret.yaml

   // 作成したsecretファイルを適用
   kubectl -n taskapp apply -f mysql-secret.yaml // -nはnamespaceを指定

   // secretの確認
   kubectl get secrets/mysql -o yaml -n taskapp
  ```

- APIサーバー用の設定ファイルのsecretを作成する
    ```
    // ~/go/src/github.com/gihyodocker/taskappで実行
    kubectl create secret generic api-config --dry-run=client -o yaml \
    --from-file=api-config.yaml=./api-config.yaml > ~/k8s/taskapp/local/api-config-secret.yaml

    // 適用 (taskapp/localで実行)
    kubectl -n taskapp apply -f api-config-secret.yaml 
    ```

# mysqlのデプロイ
- StatefulSet, serviceを作成する
