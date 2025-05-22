mkdir devise

git clone https://github.com/YouraSilin/devise.git devise

cd devise

docker compose build

docker compose run --no-deps web rails new . --force --database=postgresql --css=bootstrap

replace this files

https://github.com/YouraSilin/devise/blob/main/config/database.yml

https://github.com/YouraSilin/devise/blob/main/Dockerfile

https://github.com/YouraSilin/devise/blob/main/Gemfile

docker compose up

docker compose exec web rake db:create db:migrate

sudo chown -R $USER:$USER .

docker compose exec web rails generate devise:install

docker compose exec web rails generate devise User

docker compose exec web rails generate migration AddRoleToUsers role:string

docker compose exec web rails db:migrate

sudo chown -R $USER:$USER .


Here is a possible configuration for config/environments/development.rb:

config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

В файл app/views/layouts/application.html.erb добавьте:

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

Модифицируйте модель User (app/models/user.rb), чтобы задать роли:

class User < ApplicationRecord

\# Devise модули

devise :database_authenticatable, :registerable,

       :recoverable, :rememberable, :validatable

\# Установим роли

enum role: { viewer: 'viewer', admin: 'admin' }

\# Зададим роль по умолчанию

after_initialize do

  self.role ||= :viewer
  
end
  
end

Задайте дефолтную роль в консоли (для существующих пользователей).

docker compose exec web rails c "User.update_all(role: 'viewer')"

Создайте два контроллера — один для просмотра (режим просмотра) и другой для админки (режим редактирования).

Пример: создадим ресурс Posts.

docker compose exec web rails generate scaffold Post title:string content:text

docker compose exec web rails db:migrate

sudo chown -R $USER:$USER .

Ограничиваем доступ в контроллере:

Модифицируйте PostsController:

class PostsController < ApplicationController

  before_action :authenticate_user!
  
  before_action :authorize_admin, only: [:edit, :update, :destroy]

  \# Только администратор может редактировать и удалять
  
  private

  def authorize_admin
  
    redirect_to posts_path, alert: 'У вас нет прав для этого действия.' unless current_user&.admin?
  
  end

end

Добавьте проверку прав администратора в представления, где доступны действия редактирования и удаления:

<% if current_user&.admin? %>
  
  <%= link_to 'Редактировать', edit_post_path(@post) %>
  
  <%= button_to "Удалить эту запись", @post, method: :delete, data: { turbo_method: 'delete', turbo_confirm: "вы уверены?" } %>

<% end %>

Теперь в контроллер нужно добавить

def edit

    @post = Post.find(params[:id])
    
end

В application.html.erb нужно добавить

``` erb

<style>
      @keyframes fadeInFromNone {
        from {
          opacity: 0;
        }
        to {
          opacity: 1;
        }
      }
      .alert {
        display: none;
        padding: 10px;
        margin: 10px;
        border: 1px solid #ccc;
        border-radius: 5px;
      }
      .alert.success {
        background-color: #d4edda;
        color: #155724;
        border-color: #c3e6cb;
      }
      .alert.error {
        background-color: #f8d7da;
        color: #721c24;
        border-color: #f5c6cb;
      }
    </style>

    <% flash.each do |type, message| %>
      <% css_class = type == "notice" ? "success" : "error" %> <!-- Определение типа уведомления -->
      <div class="alert <%= css_class %>">
        <%= message %>
      </div>
    <% end %>

    <script type="module">
      document.addEventListener("turbo:load", () => {
        // Показываем уведомления
        const alerts = document.querySelectorAll(".alert");
        alerts.forEach((alert) => {
          alert.style.display = "block";
          alert.style.opacity = "1";
          alert.style.animation = "fadeInFromNone 1.2s";

          // Автоматическое скрытие через 2 секунды
          setTimeout(() => {
            alert.style.transition = "opacity 1.2s";
            alert.style.opacity = "0";

            // Удаляем элемент после того, как он полностью исчезнет
            setTimeout(() => {
              alert.style.display = "none";
            }, 1200);
          }, 2000);
        });
      });
    </script>

    <script type="module">
      document.addEventListener("turbo:load", () => {
        // Показываем уведомления
        const alerts = document.querySelectorAll(".alert");
        alerts.forEach((alert) => {
          alert.style.display = "block";
          alert.style.opacity = "1";
          alert.style.animation = "fadeInFromNone 1.2s";

          // Автоматическое скрытие через 2 секунды
          setTimeout(() => {
            alert.style.transition = "opacity 1.2s";
            alert.style.opacity = "0";

            // Удаляем элемент после того, как он полностью исчезнет
            setTimeout(() => {
              alert.style.display = "none";
            }, 1200);
          }, 2000);
        });
      });
    </script>
    
```
    
Добавление администратоа

docker compose exec web rails c

Найдите пользователя, который должен стать администратором:

user = User.find_by(email: "email@example.com")

Установите ему роль admin:

user.update(role: "admin")
