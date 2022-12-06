In simple terms, Query Object allows you to encapsulate complex database queries. 

**Why do we need it and what problems can this pattern solve?**

Sometimes we have very complex queries that are used directly in the business logic. For example, the following query can be used many times in different controllers and service objects:

```rb
def index
  seasons = Season.joins(league: :country).where("countries.name = 'England'")
  
  render json: seasons
end
```

**What problem does this create for us?**

- It's impossible to test separately from the controller. 
- It's very difficult to stub/mock db request 
- It's not DRY
- It's not clear, hard to read and understand what's going on.

**So what can we do to fix it?**
- We can try to move this logic to the model level.
- We can try to create a Query Object and move that logic there.

If we decide to choose the first option and use a model, there are 2 ways to implement this:

## Class methods
```rb
# frozen_string_literal: true

class Season < ApplicationRecord

  def self.by_league(league)
    where(league: league)
  end

  def self.by_status(status)
    where(status: status)
  end

end

Season.by_league(league)
Season.by_status(status)
```

## Scope

```rb
class Season < ApplicationRecord

  scope :by_league, -> (league) { where(league: league) }
  scope :by_status, -> (status) { where(status: status) }

end

Season.by_league(league)
Season.by_status(status)
```

The interface looks good, but we still have some problems. _The most important one is the violation of Single responsibility principle._ The model becomes too big and fat. We can get more than 100 methods in a year. So it will be very difficult to read, change, and maintain. It would be nice to have the same interface, but move that logic into a separate class.

**How can we do that?**

The Query Object pattern was created to solve this problem. 

## Let's start with the simplest Query Object. 

```rb
# app/units/queries/season.rb

module Queries
  class Season

    class << self
      def by_league(league)
        ::Season.where(league: league)
      end

      def by_status(status)
        ::Season.where(status: status)
      end
    end

  end
end

Queries::Season.by_league(league)
Queries::Season.by_status(status)
```

Interface looks fine, but there's one important problem - methods are not chainable  and we have to repeat `::Season` prefix every time. For example, if you want to write something like this `Queries::Season.by_league(league).by_status(status)`, it won't work.

**So, how can we fix it?**

Rails has an interesting method - [extending](https://apidock.com/rails/ActiveRecord/QueryMethods/extending). This method allows you to add any methods on the model and chain them. For example, the following works fine.

```rb
module Scopes
  def by_league(league)
    where(league: league)
  end

  def by_status(status)
    where(status: status)
  end
end

query = Season.all.extending(Scopes)
query.by_league(league).by_status(status)

```

So now we need to somehow delegate methods from `Queries::Season` to `Season.all.extending(Scopes)`. How can we do this?
Rails provides a method [delegate_missing_to](https://apidock.com/rails/v5.2.3/Module/delegate_missing_to) 
So we can simply delegate all methods to be called on `Queries::Season` to `Season.all.extending(Scopes)`.

```rb
# frozen_string_literal: true

module Queries
  class Season

    module Scopes
      def by_league(league)
        where(league: league)
      end

      def by_status(status)
        where(status: status)
      end
    end

    class << self
      delegate_missing_to :relation

      def relation
        ::Season.all.extending(Scopes)
      end
    end

  end
end

Queries::Season.by_league(league).by_status(status)
```

So now all methods are chainable. The last problem we can solve is that the code is NOT DRY, and we don't want to repeat the following every time:

```rb
    class << self
      delegate_missing_to :relation

      def relation
        ::Season.all.extending(Scopes)
      end
    end
```

Let's create a parent class `Query` and move the common logic there.

```rb
# lib/query.rb

class Query

  class << self

    attr_reader :model

    def set_model(model)
      @model = model
    end

    delegate_missing_to :relation

    private

    def relation
      model.all.extending(self::Scopes)
    end

  end

end
```

And let's inherit from this class:

```rb
# app/units/queries/season.rb

module Queries
  class Season < Query

    set_model ::Season

    module Scopes
      def by_league(league)
        where(league: league)
      end

      def by_status(status)
        where(status: status)
      end
    end

  end
end

Queries::Season.by_league(league).by_status(status)
```

**So the final solution may look as follows:**

```rb
def index
  seasons = Queries::Season.by_country('England')
  
  render json: seasons
end
```

## Conclusion:
- Query Object can be easily tested separately from the controller
- Query Object can be easily stub/mocked
```rb
allow(Queries::Season).to receive(:by_country).with('England').and_return(...)
```
- Query Object is DRY and reusable
- The code is clear, easy to read and understand 
- Query Object is separated from the model and allows us to avoid FAT model
