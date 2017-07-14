# Rationales

This document contains what are at times lengthy rationales and justifications
for the decisions made in the main [Style guide](./README.md).

## Table of Contents
  1.  [Private Method Indentation](#private-method-indentation)
  1.  [Line Length](#line-length)
  1.  [Callbacks](#callbacks)

### Private Method Indentation

Having more indentation on private methods does not improve the overall readability of the file.
```
class Foo

  class NestedClass
    def nested_method
    end
  end

  ...
  private
    def bar
    end
end
```

When a nested class is present, the public methods of that nested class has the same indentation
as the private method which can be confusing.

### Line Length

Keeping code visually grouped together (as a 100-character line limit enforces)
makes it easier to understand. For example, you don't have to scroll back and
forth on one line to see what's going on -- you can view it all together.

Here are examples from our codebase showing several techniques for
breaking complex statements into multiple lines that are all < 100
characters. Notice techniques like:

* liberal use of linebreaks inside unclosed `(` `{` `[`
* chaining methods, ending unfinished chains with a `.`
* composing long strings by putting strings next to each other, separated
  by a backslash-then-newline.
* breaking long logical statements with linebreaks after operators like
  `&&` and `||`

```ruby
scope = Translation::Phrase.includes(:phrase_translations).
  joins(:phrase_screenshots).
  where(:phrase_screenshots => {
    :controller => controller_name,
    :action => JAROMIR_JAGR_SALUTE,
  })
```

```ruby
translation = FactoryGirl.create(
  :phrase_translation,
  :locale => :is,
  :phrase => phrase,
  :key => 'phone_number_not_revealed_time_zone',
  :value => 'Símanúmerið þitt verður ekki birt. Það er aðeins hægt að hringja á '\
            'milli 9:00 og 21:00 %{time_zone}.'
)
```

```ruby
if @reservation_alteration.checkin == @reservation.start_date &&
   @reservation_alteration.checkout == (@reservation.start_date + @reservation.nights)

  redirect_to_alteration @reservation_alteration
end
```

```erb
<% if @presenter.guest_visa_russia? %>
  <%= icon_tile_for(I18n.t("email.reservation_confirmed_guest.visa.details_header",
                           :default => "Visa for foreign Travelers"),
                    :beveled_big_icon => "stamp") do %>
    <%= I18n.t("email.reservation_confirmed_guest.visa.russia.details_copy",
               :default => "Foreign guests travelling to Russia may need to obtain a visa...") %>
  <% end %>
<% end %>
```

These code snippets are very much more readable than the alternative:

```ruby
scope = Translation::Phrase.includes(:phrase_translations).joins(:phrase_screenshots).where(:phrase_screenshots => { :controller => controller_name, :action => JAROMIR_JAGR_SALUTE })

translation = FactoryGirl.create(:phrase_translation, :locale => :is, :phrase => phrase, :key => 'phone_number_not_revealed_time_zone', :value => 'Símanúmerið þitt verður ekki birt. Það er aðeins hægt að hringja á milli 9:00 og 21:00 %{time_zone}.')

if @reservation_alteration.checkin == @reservation.start_date && @reservation_alteration.checkout == (@reservation.start_date + @reservation.nights)
  redirect_to_alteration @reservation_alteration
end
```

```erb
<% if @presenter.guest_visa_russia? %>
  <%= icon_tile_for(I18n.t("email.reservation_confirmed_guest.visa.details_header", :default => "Visa for foreign Travelers"), :beveled_big_icon => "stamp") do %>
    <%= I18n.t("email.reservation_confirmed_guest.visa.russia.details_copy", :default => "Foreign guests travelling to Russia may need to obtain a visa prior to...") %>
  <% end %>
<% end %>
```

### Callbacks

Using callbacks tend to make the test slower. This is usually a bad practice when it
does something other than touch the model layer. This includes creating necessary associations and
also updating values to its own. Having a callback that affects other models will
result to a test that will be hard to write since it would be necessary to create the other models.

```ruby
# bad example
class User < ActiveRecord::Base
  has_one :address

  after_save :ensure_correct_address!

  private
  def ensure_correct_address!
    ...
  end
end

class Address < ActiveRecord::Base
end
```

Testing this, would require both instances to be present.

```ruby
# good example
class User < ActiveRecord::Base
  after_create :create_address!
end

class Address < ActiveRecord::Base
end
```

With this example, a user always has an address. Having a callback in this case makes
sense since this ensures consistency in our data.

Another case where a callback can be harmful is when it sends out data that would be
required to be stubbed in tests.

```
# bad example
class User < ActiveRecord::Base
  after_create :create_s3_bucket!

  private
  def create_s3_bucket!
    ...
  end
end
```

All the tests should fail in this case because it is calling out to an outside service. The `User`
should not care if it has an `s3` bucket or not since it is outside of its domain.
