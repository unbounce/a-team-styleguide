# Rails Style Guide

## Table of Contents

  1. [Models](#models)
    1. [Callbacks](#callbacks)
    1. [Scopes](#scopes)
  1. [Controllers](#controllers)
    1. [Returns on Renders](#returns-on-renders)


## Models
### Callbacks
* <a name="callbacks"></a>Only use callbacks that touches the model layer
    ([rationale](./rationales.md/#callbacks))
    <sup>[[link](#callbacks)]</sup>
    ```ruby
    # bad
    class Newsletter
      after_create :send_email_to_subscribers
    end

    # good
    class Person
      after_create :create_address!
    end
    ```

### Scopes
* <a name="scope-lambda"></a>When defining ActiveRecord model scopes, wrap the
    relation in a `lambda`.  A naked relation forces a database connection to be
    established at class load time (instance startup).
    <sup>[[link](#scope-lambda)]</sup>

    ```ruby
    # bad
    scope :foo, where(:bar => 1)

    # good
    scope :foo, -> { where(:bar => 1) }
    ```

## Controllers
### Returns on Renders
* <a name="next-line-return"></a>When immediately returning after calling
    `render` or `redirect_to`, put `return` on the next line, not the same line.
    <sup>[[link](#next-line-return)]</sup>

    ```ruby
    # bad
    render :text => 'Howdy' and return

    # good
    render :text => 'Howdy'
    return

    # still bad
    render :text => 'Howdy' and return if foo.present?

    # good
    if foo.present?
      render :text => 'Howdy'
      return
    end
    ```
