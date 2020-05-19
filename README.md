# ① myappをユーザー直下に配置します
<br>

docker-hands-on-master.zipを解凍し、myappディレクトリをユーザー直下に配置します
![キャプチャ](https://user-images.githubusercontent.com/53431136/69335083-7033d580-0c9f-11ea-9253-b1a876eae9ae.PNG)

<br>


![image-20191119101214129](https://user-images.githubusercontent.com/53431136/69317135-a5c7c700-0c7d-11ea-9b27-40e50614be1f.png)
<br><br>
# ② 配置したmyappフォルダに移動します
```
cd C:\Users\myapp
```
<br><br>
# ③ Railsの新規プロジェクトを作成します
```
docker-compose run web rails new . --force --no-deps --database=mysql
```
→実行するとmyapp配下にファイルが増えます
<br><br>
# ④ DBの設定を変更するため、一度コンテナを停止します
```
docker-compose down
```
<br><br>
# ⑤ DBの向きを変更します
C:\Users\myapp\config\database.yml　を配布したファイルでファイルごと上書きします
<br><br>
# ⑥ ファイルを配置します
C:\Users\myapp\mysql-confd　に　ファイル（default_authentication.cnf）を配置します
<br><br>
# ⑦ イメージをビルドします
```
docker-compose build
```
→「Successfully built ●●●●●●●●●」が出たら完了！
<br><br>
# ⑧ 先にDBを起動します
```
docker-compose up -d db
```
<br><br>
# ⑨ Railsのサービスを起動します
```
docker-compose run web rake db:create
```
<br><br>
# ⑩ ビルドしたイメージからコンテナを起動します 
```
docker-compose up -d
```
<br><br>
<br>
# ★チェックポイント★
正常に起動しているかを確認
```
docker ps -a
```
↑のように「myapp_web」と「mysql」のコンテナがupしていればOK！
<br><br>

以下に接続してみましょう！
<br><br>
```
http://localhost:3000/
```
![rails](https://user-images.githubusercontent.com/53431136/69325441-85076d80-0c8d-11ea-9610-786f056a8201.png)


<br><br>

# ⑪ WEBのコンテナに接続します
```
docker exec -it myapp_web_1 bash
```

<br><br>
# ⑫ 以下を、１行ずつ実行します
```
rails g model Employee number:string name:string date:string
```
```
rails db:migrate
```
```
rails g controller Employees
```
<br><br>
# ⑬ myapp\app\controllers　配下の以下のファイルを編集します
対象ファイル：employees_controller.rb
```
class EmployeesController < ApplicationController
  def index
    @employees = Employee.all
  end
end
```
<br><br>
# ⑭ myapp\db　配下の以下のファイルを編集します
対象ファイル：seeds.rb
```
Employee.create(
  [
    {
      number: '001',
      name: 'Yamada',
      date: '2018/01/01',
    },
    {
      number: '002',
      name: 'Tanaka',
      date: '2019/04/01',
    },
    {
      number: '003',
      name: 'Sato',
      date: '2019/05/01',
    },
  ],
)
```
<br><br>
# ⑮ ⑭のファイルを元にデータを作成します
```
rails db:seed
```
<br><br>
# ⑯ myapp\config　配下の以下のファイルを編集します
対象ファイル：routes.rb
```
Rails.application.routes.draw do
  root to: 'employees#index'
  resources :employees
end
```
<br><br>
# ⑰ 以下のコマンドで画面を作成します
```
touch app/views/employees/index.html.erb
```

<br><br>
# ⑱ app/views/employees　配下の以下のファイルを編集します
対象ファイル：index.html.erb
```
<h1>List of employees</h1>

<table border="1" width="700" cellspacing="0" cellpadding="5" bordercolor="#333333" class="table table-hover">
  <thead class="thead-dark">
    <tr>
      <th bgcolor="#87cefa" class="align-middle" scope="col" width="100"><font size="+1" color="#FFFFFF">Employee no</font></th>
      <th bgcolor="#87cefa" class="align-middle" scope="col" width="400"><font size="+1" color="#FFFFFF">Name</font></th>
      <th bgcolor="#87cefa" class="align-middle" scope="col" width="150"><font size="+1" color="#FFFFFF">Hire date</font></th>
    </tr>
  </thead>


  <tbody>
    <% @employees.each do |employee| %>
      <tr scope="row">
        <td><%= employee.number %></td>
        <td><%= employee.name %></td>
        <td><%= employee.date %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```
<br><br>
# ⑲ 以下に接続して確認！
```
http://localhost:3000/
```
![image-20191121003744232](https://user-images.githubusercontent.com/53431136/69325948-53db6d00-0c8e-11ea-982e-650f9e31d71d.png)

<br>
このような画面が表示されたら完成！


<br><br>
<br>
# ★Dockerイメージをクラウド（Azure）にアップロードしよう★
<br><br>
# ⑳ DBの向き先をAzureのDBに変更するため
C:\Users\azure_db\database.yml　を
C:\Users\myapp\config\database.yml　にファイルごと上書きします

<br><br>
# ㉑ Dockerファイルの最後に以下を追記します
```
CMD /bin/sh -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 8080 -b '0.0.0.0'"
```
<br><br>
# ㉒ 以下のコマンドでDcokerのビルドします
```
docker-compose build --no-cache
```
<br><br>
# ㉓ 以下のコマンドでAzureに接続します
※パスワードが求められます
```
docker login azureContainerRegistryName1.azurecr.io --username azureContainerRegistryName1
```
<br><br>
# ㉔ 以下のコマンドでDcokerイメージにタグ付けをします

```
docker tag myapp_web azureContainerRegistryName1.azurecr.io/myapp_web:v1.0.0
```
<br><br>
# ㉒ 以下のコマンドでDockerイメージをAzureにデプロイします
```
docker push azureContainerRegistryName1.azurecr.io/myapp_web:v1.0.0
```
<br><br>
# ㉓ 以下に接続してみましょう！
```
https://webapl1001.azurewebsites.net/
```