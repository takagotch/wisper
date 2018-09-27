### wisper
---

https://github.com/krisleech/wisper

```sh
gem 'wisper', '2.0.0'

bundle exec rspec
ls **/*.rb | entr bundle exec rspec
```

```
class CancelOrder
  include Wisper::Publisher
  def call(order_id)
    order = Order.find_by_id(order_id)
    if order.cancelled?
      broadcast(:cancel_order_successful, order.id)
    else
      broadcast(:cancel_order_failed, order.id)
    end
  end
end

cancel_order = CancelOrder.new
cancel_order.subscribe(OrderNotifier.new)
cancel_order.call(order_id)

class OrderNotifier
  def cancel_order_successful(order_id)
    order = Order.find_by_id(order_id)
  end
end

cancel_order = CancelOrder.new
cancel_order.on(:cancel_order_successful) { |order_id| ... }
            .on(:cancel_order_failed) { |order_id| ... }
cancel_order.call(order_id)


cancel_order = CancelOrder.new
cancel_order.on() {}
            .on() {}
cancel_order.call(order_id)

cancel_order.subscribe(OrderNotifier.new, async: true)



class OrderNotifier
  def self.cancel_order_successful(order_id)
    order = Order.find_by_id(order_id)
  end
end


class CancelOrderController < ApplicationController
  def create
    cancel_order = CancelOrder.new
    
    cancel_order.subscribe(OrderMailer, async: true)
    cancel_order.subscribe(ActivityRecorder, async: true)
    cancel_order.subscribe(StatisticsRecorder, async: true)
    
    cancel_order.on(:cancel_order_successful) { |order_id| redrect_to order_path(order_id) }
    cancel_order.on(:cancel_order_failed) { |order_id| render action: :new}
    
    cancel_order.call(order_id)
  end
end

class Order < ActiveRecord::Base
  include Wisper::Publisher
  
  after_commit :publish_creation_successful, on: :create
  after_validation :publish_creation_failed, on: create
  private
  def publish_creation_successful
    broadcast(:order_creation_successful, self)
  end
  def publish_creation_failed
    broadcast(:order_creation_failed, self) if errors.any?
  end
end

Wisper.subscribe(MyListener.new)

Wisper.subscribe(MyListener.new, scope: :MyPublisher)
Wisper.subscribe(MyListener.new, scope: MyPublisher)
Wisper.subscribe(MyListener.new, scope: "MyPublisher")
Wisper.subscribe(MyListener.new, scope: [:MyPublisher, :MyOtherPublisher])

MyPublisher.subscribe(MyListener.new)

Wisper.subscribe(MyListener.new, OtherListener.new) do
end

post_creator.subscribe(PusherListener.new, on: :create_post_successful)

post_creator.subscribe(PubherListener.new, prefix: :on)

report_creator.subscribe(MailResponder.new, with: :successful)

report_creator.subscribe(MailResponder.new, on: :create_report_successful,
                                            with: :successful)
report_creator.subscribe(MailResponder.new, on: :create_report_failed,
                                            with: :failed)

after { Wisper.clear }

```

