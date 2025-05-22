mkdir devise

git clone https://github.com/YouraSilin/jquery.git devise

cd devise

docker compose build

docker compose run --no-deps web rails new . --force --database=postgresql --css=bootstrap

replace this files

https://github.com/YouraSilin/jquery/blob/main/config/database.yml

https://github.com/YouraSilin/jquery/blob/main/Dockerfile

https://github.com/YouraSilin/jquery/blob/main/Gemfile

docker-compose up --remove-orphans

docker compose exec web rake db:create db:migrate

sudo chown -R $USER:$USER .

docker compose exec web rails generate devise:install

docker compose exec web rails generate devise User

docker compose exec web rails generate migration AddRoleToUsers role:string

docker compose exec web rails db:migrate

sudo chown -R $USER:$USER .

Here is a possible configuration for config/environments/development.rb:

```erb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```
