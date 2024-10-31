# Hướng Dẫn congif Gem `jwt` với Gem Devise Trong Rails

Hướng dẫn chi tiết từng bước để tích hợp và cấu hình gem `jwt` với gem `deivse` cho xác thực bằng JSON Web Token (JWT) trong ứng dụng Ruby on Rails.

## 1. Thêm `jwt` và `devise` vào Gemfile
- Mở file `Gemfile`, thêm dòng sau để thêm gem `jwt` và gem `devise` vào dự án:

  ```ruby
  gem 'jwt'
  gem 'devíe'
- Sau đó chạy lệnh
  ```ruby
  bundle install
## 2. Tạo File Khởi Tạo Cho JWT
- Trong thư mục `config/initializers`, tạo một file mới có tên `jwt_strategy.rb` để lưu trữ khóa bí mật cho JWT:
  ```ruby
  
  module Devise
    module Strategies
      class JwtStrategy < Base
        def valid?
          request.headers["Authorization"].present?
        end
  
        def authenticate!
          raise ExceptionHandler::AuthenticationError if no_payload_or_payload_user_id
  
          success! User.find_by(id: payload["user_id"], access_token: access_token)
        end
  
        private
  
        def access_token
          bearer_token = request.headers.fetch("Authorization", "").split
          return nil unless (bearer_token.first || "").casecmp("bearer").zero?
          bearer_token.last
        end
  
        def no_payload_or_payload_user_id
          !payload || !payload.key?("user_id")
        end
  
        def payload
          JsonWebToken.decode(access_token)
        rescue JWT::ExpiredSignature
          raise ExceptionHandler::TokenTimeoutError
        rescue StandardError
          nil
        end
      end
    end
  end

## 3. Tạo Service Mã Hóa và Giải Mã JWT
  - Tạo folder `app/lib/json_web_token.rb`.
    ```ruby
      class JsonWebToken
        HMAC_SECRET = ENV.fetch('SECRET_KEY_BASE', nil).freeze
      
        class << self
          def encode(payload, exp = 30.minutes.from_now)
            payload[:exp] = exp.to_i
            ::JWT.encode(payload, HMAC_SECRET)
          end
      
          def decode(token)
            body = ::JWT.decode(token, HMAC_SECRET)[0]
            ActiveSupport::HashWithIndifferentAccess.new body
          end
      
          def valid?(payload)
            Time.zone.at(payload['exp']) > Time.zone.now
          end
        end
      end

## 3. Config gem devise với jwt
- Thêm đoạn config vào trong file `devise.rb`
  ```ruby
    config.warden do |manager|
      manager.strategies.add(:jwt, Devise::Strategies::JwtStrategy)
      manager.default_strategies(scope: :user).unshift :jwt
    end
## 4. Tạo chức năng login dùng devise với jwt trong  `SessionsController`
  - Có thể sử dụng file `SessionsController` khi cài `gem devise` để tạo chức năng login `app/controllers/users/sessions_controller.rb`
    ```ruby
      module Users
      class SessionsController < Devise::SessionsController
        before_action :configure_permitted_parameters
    
        respond_to :json
    
        def create
          resource = warden.authenticate!(auth_options.merge(store: false, recall: "#{controller_path}#failure"))
          access_token = JsonWebToken.encode(user_id: resource.id)
          render json: {
            access_token: access_token
          }, status: :ok
        end
    
        def failure
          render json: { error: 'email_password_invalid' }, status: :unauthorized
        end
    
        private
    
        def configure_permitted_parameters
          devise_parameter_sanitizer.permit(:sign_in) { |u| u.permit(:email, :password) }
        end
      end
    end
## Kết.
- Với các bước trên ta đã config và chạy chức login bằng phương pháp thiết lập xác thực dựa trên token trong ứng dụng Rail thông qua gem jwt và gem devise



