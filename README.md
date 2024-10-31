# Hướng Dẫn Cấu Hình Gem `jwt` Trong Dự Án Rails

Hướng dẫn chi tiết từng bước để tích hợp và cấu hình gem `jwt` cho xác thực bằng JSON Web Token (JWT) trong ứng dụng Ruby on Rails.

## 1. Thêm `jwt` vào Gemfile
- Mở file `Gemfile`, thêm dòng sau để thêm gem `jwt` vào dự án:

  ```ruby
  gem 'jwt'
- Sau đó chạy lệnh
  ```ruby
  bundle install
## 2. Tạo File Khởi Tạo Cho JWT
- Trong thư mục `config/initializers`, tạo một file mới có tên `jwt.rb` để lưu trữ khóa bí mật cho JWT:
  ```ruby
  # config/initializers/jwt.rb
  
  # Khóa bí mật dùng để mã hóa JWT. (Lưu ý: không lưu trữ khóa bí mật ở đây trong môi trường production, thay vào đó hãy dùng biến môi trường)
  JWT_SECRET = ENV['JWT_SECRET'] || 'your_default_secret_key'
## 3. Tạo Service Mã Hóa và Giải Mã JWT
- Trong thư mục `lib`, tạo file `json_web_token.rb` với các phương thức mã hóa và giải mã JWT:
  ```ruby
  # lib/json_web_token.rb
  
  require 'jwt'
  
  class JsonWebToken
    SECRET_KEY = JWT_SECRET
  
    # Phương thức để mã hóa JWT với payload và thời hạn
    def self.encode(payload, exp = 24.hours.from_now)
      payload[:exp] = exp.to_i
      JWT.encode(payload, SECRET_KEY)
    end
  
    # Phương thức để giải mã JWT
    def self.decode(token)
      decoded = JWT.decode(token, SECRET_KEY)[0]
      HashWithIndifferentAccess.new(decoded)
    rescue JWT::DecodeError
      nil
    end
  end

- Lưu ý: Đảm bảo thư mục lib được tự động load trong Rails bằng cách thêm dòng sau vào file `config/application.rb` nếu cần:
  ```ruby
  config.autoload_paths << Rails.root.join('lib')
## 4. Tạo Controller Đăng Nhập
- Tạo một controller `AuthController` để xử lý đăng nhập và trả về JWT:
  ```ruby
  # app/controllers/auth_controller.rb

  class AuthController < ApplicationController
    def login
      user = User.find_by(email: params[:email])
  
      if user&.authenticate(params[:password])
        token = JsonWebToken.encode(user_id: user.id)
        render json: { token: token }, status: :ok
      else
        render json: { error: 'Invalid email or password' }, status: :unauthorized
      end
    end
  end
## 5. Thêm Xác Thực Cho Các Controller Khác
- Trong `ApplicationController`, thêm phương thức `authorize_request` để kiểm tra JWT cho các request cần xác thực:
  ```ruby
  # app/controllers/application_controller.rb
  
  class ApplicationController < ActionController::API
    before_action :authorize_request
  
    private
  
    def authorize_request
      header = request.headers['Authorization']
      header = header.split(' ').last if header
      decoded = JsonWebToken.decode(header)
      @current_user = User.find(decoded[:user_id]) if decoded
    rescue ActiveRecord::RecordNotFound, JWT::DecodeError
      render json: { errors: 'Unauthorized' }, status: :unauthorized
    end
  end
## 6. Bảo Mật Khóa Bí Mật JWT Bằng Biến Môi Trường
- Để bảo mật, hãy lưu khóa bí mật JWT trong biến môi trường thay vì viết trực tiếp vào mã nguồn, đặc biệt là trong môi trường production. Thêm biến `JWT_SECRET` vào file `.env`:
  ```ruby
  JWT_SECRET=your_production_secret_key



