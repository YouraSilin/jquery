```bash

mkdir devise

git clone https://github.com/YouraSilin/jquery.git devise

cd devise

docker compose build

docker compose run --no-deps web rails new . --force --database=postgresql --css=bootstrap

```

replace this files

https://github.com/YouraSilin/jquery/blob/main/config/database.yml

https://github.com/YouraSilin/jquery/blob/main/Dockerfile

https://github.com/YouraSilin/jquery/blob/main/Gemfile

```bash

docker-compose up --remove-orphans

docker compose exec web rake db:create db:migrate

sudo chown -R $USER:$USER .

docker compose exec web rails generate devise:install

docker compose exec web rails generate devise User

docker compose exec web rails generate migration AddRoleToUsers role:string

docker compose exec web rails db:migrate

sudo chown -R $USER:$USER .

```

Here is a possible configuration for config/environments/development.rb:

```erb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

В файл app/views/layouts/application.html.erb добавьте:

```erb
<ul class="navbar-nav ms-auto">
  <% if user_signed_in? %>
    <li class="nav-item">
      <span class="nav-link">Вы зашли как: <strong><%= current_user.email %></strong></span>
    </li>
    <li class="nav-item">
      <%= button_to "Выйти", destroy_user_session_path, method: :delete, class: "nav-link" %>
    </li>
  <% else %>
    <li class="nav-item">
      <%= link_to "Войти", new_user_session_path, class: "nav-link" %>
    </li>
    <li class="nav-item">
      <%= link_to "Регистрация", new_user_registration_path, class: "nav-link" %>
    </li>
  <% end %>
</ul>
<% if flash[:alert] %>
  <div class="alert alert-danger">
    <%= flash[:alert] %>
  </div>
<% end %>

```

Модифицируйте модель User (app/models/user.rb), чтобы задать роли:

```erb
class User < ApplicationRecord

# Devise модули

devise :database_authenticatable, :registerable,

   :recoverable, :rememberable, :validatable

# Установим роли

enum role: { viewer: 'viewer', admin: 'admin' }

# Зададим роль по умолчанию

after_initialize do

self.role ||= :viewer

end

end

```

Задайте дефолтную роль в консоли (для существующих пользователей).

```bash

docker compose exec web rails c "User.update_all(role: 'viewer')"

```

Создайте два контроллера — один для просмотра (режим просмотра) и другой для админки (режим редактирования).

Пример: создадим ресурс Posts.

```bash

docker compose exec web rails generate scaffold Post title:string content:text

docker compose exec web rails db:migrate

sudo chown -R $USER:$USER .

```



Ограничиваем доступ в контроллере:

Модифицируйте PostsController:

```erb

class PostsController < ApplicationController

  before_action :authenticate_user!
  
  before_action :authorize_admin, only: [:edit, :update, :destroy]
  
  # Только администратор может редактировать и удалять
  
  def edit
    @post = Post.find(params[:id])
  end
  
  private
  
  def authorize_admin
    redirect_to posts_path, alert: 'У вас нет прав для этого действия.' unless current_user&.admin?
  end
  
end

```

Добавьте проверку прав администратора в представления, где доступны действия редактирования и удаления:

```erb

<% if current_user&.admin? %>

  <%= link_to 'Редактировать', edit_post_path(@post) %>

  <%= button_to "Удалить эту запись", @post, method: :delete, data: { turbo_method: 'delete', turbo_confirm: "вы уверены?" } %>

<% end %>

```

Добавление администратоа

```erb

docker compose exec web rails c
user = User.find_by(email: "email@example.com")
user.update(role: "admin")

```

Импорт Jquery

```bash

docker compose exec web bin/importmap pin jquery

```

В importmaps.rb должно появиться

```erb

pin "jquery", to: "https://cdn.jsdelivr.net/npm/jquery@3.6.0/dist/jquery.min.js", preload: true

```

В application.js нужно добавить

```erb

import $ from "jquery";

$(document).ready(function() {
  console.log("jQuery is ready!");
});

```
