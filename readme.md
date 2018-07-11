# 180710

* current_user : 몇몇개의 gem에서 정의된다. ex)Devise
* login 후에 세션에 user_id를 저장하게된다. 

```ruby
class ApplicationController < ActionController::Base
  def current_user
    return unless session[:user_id]
    @current_user ||= User.find(session[:user_id])
  end
end
```



1. 부트스트랩 버전 확인

2. 설치된 gem들을 사용할 수 있게 설정하는거

3. 우리가 사용할 템플릿 파일에서 사용하는 stylesheet파일들을 확인하고 `vendor/aseests/stylesheet`에 복사한다. 

   > vendor 폴더를 사용하는 이유는 여기에 들어가는 css와 js는 거의 변화가 없는 library정도에 해당하는 파일이 들어간다.  변화할 파일들(custom.css/style.css)은 app/assets/stylesheet에 넣어둔다.

4. `app/assets/stylesheet/application.css`-> `scss`로 확장자를 바꾸고 우리가 `vendor`에 넣어둔 파일들을 전부 import한다. 기존에 있던 *= 형태로 되어 있는 import는 전부 제거한다. 그 다음 @import로 전부 import해줘야 한다.

5. 동일한 형태로 js도 진행한다.(파일복사) 새로운 컨트롤러가 만들어질 때 `.coffee`로 적용되는데 이 확장자를 `.js`로 바꾸준다.  (js2coffee로 바꿔줘도 됨.) . //= require-tree . 는 삭제하고 application.js에서는 bootstrap과 jquery 혹은 모든 페이지에서 공통되는 js만 import한다.

6. `config/initializers/assets.rb`에서  수정하고, 꼭 서버 restart

```ruby
#주석처리를 해제하고 우리가 사용할 컨트롤러에 해당하는 js,scss파일명을 나열한다.
Rails.application.config.assets.precompile += %w( home.js 
                                                  home.scss
                                                  food_datails.js
                                                  food_datails.scss)
```

7. rake assets:precompile을  실행해서 scss파일과 js파일에 이상이 없는지 확인한다.

   (<-> rake assets:clobber) 이상이 있는 부분은 css,js에 맞춰서 수정한다.

8.  이제 실제 body에 해당하는 부분을 우리 페이지로 가져오면 되는데, nav,footer는 파일을 분리하는 것이 좋다.  반복적으로 사용될 친구들인데  render(partial)을 이용해서 view를 분리하는게 좋다. 그래서 필요한 부분에 가져다 사용하면 된다. 

9. 실제로 우리가 만든 view에는 우리 서비스가 제공되는 페이지가 들어간다.

10. js의 경우에는 대부분 문서 제일 마지막에 들어가는데 이부분을 해결하기 위해서 `yield 'content_name'`과 `content_for'content_name'`과 같은 전략을 사용한다. 

(content_for : 실제 사용되는 로직에 관한 js가 들어있다. 페이지가 끝날 때, 문서 하단으로 들어간다.)

11. 우리가 1~5번까지 작성했던 js 파일과 scss파일을 실제 뷰에서 사용하기 위해서 `stylesheet_link_tag`와 `javascript_include_tag`에 각 컨트롤러에 맞는 파일을 가져오기 위해서 `params[:controller]`라는 매개변수를 줘서 각 컨트롤러마다 다른 scss와 js가 적용되도록한다.
12. 이 모든 것을 `assets_pipeline`이라고 하는데, 이는 페이지를 더 빠르게 로드하기 위한 전략으로 사용된다. 

=>자세한 내용은 공식문서 참조: http://guides.rubyonrails.org/asset_pipeline.html



## Pusher

* CUD Chatroom (Create/Update/Delete)
* Join : 내가 방에 들어갔을 때, 채팅 시작
* Chat : 실제 채팅

`Gemfile`

```ruby
#pusher
gem 'pusher'

#authentication
gem 'devise'

#key encrypt
gem 'figaro'

#turbolink삭제
```

```ruby
$ rails g devise:install
$ rails g devise users			#user
$ rails g scaffold chat_room	#room
$ rails g model chat 			#chat
$ rails g model admission		#admission join table
```

* counter cache

`admission.rb`

```ruby
class Admission < ApplicationRecord
    belongs_to :user
    belongs_to :chat_room,counter_cache: :true		#현재 인원수 저장
end
```

`create_chat_rooms.rb`

```ruby
class CreateChatRooms < ActiveRecord::Migration[5.0]
  def change
    create_table :chat_rooms do |t|
      t.string    :title
      t.string    :master_id
      
      t.integer   :max_count 
      t.integer   :admissions_count, default: 0
        
      t.timestamps
    end
  end
end
```



* references

`schema.rb`

```ruby
   t.integer  "chat_room_id"		#t.references 	:chat_room
   t..integer  "user_id"			#t.references 	:user
```



1. 'join'은 메소드가 'post' . user_admit_room 이라는 메소는 `chat_room.rb` 
2. Admissions.create
3. user_join_chat_room_notification이 동작. => pusher한테 chat_room이라는 채널에 join이라는 이벤트가 발생한걸 알려준다. data는 chat_room_id. 
4. subscribe하는 친구가 인덱스에 인원수를 증가시킨다.

### 과제

1. max_count 안 넘도록
2. 이미 참여한 사람은 join버튼이 안보이게 
3.   padding: 25px 0;
     display: inline-block;

## wysiwyg editor 

* summernote