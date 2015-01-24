gemoji
======

Emoji images and names. See the LICENSE for copyright information.


Installation
------------

Add `gemoji` to your Gemfile.

``` ruby
gem 'gemoji'
```

**Sync images**

Images can be copied to your public directory with `rake emoji` in your app. This is the recommended approach since the images will be available at a consistent location. This works best with cached formatted user content generated by tools like [html-pipeline](https://github.com/jch/html-pipeline).

``` ruby
# Rakefile
load 'tasks/emoji.rake'
```

```
$ rake emoji
```

**Assets Precompiling**

If you must, you can manually add all the images to your asset load path.

``` ruby
# config/application.rb
config.assets.paths << Emoji.images_path
```

Then have them compiled to public on deploy.

``` ruby
# config/application.rb
config.assets.precompile << "emoji/**/*.png"
```

**WARNING** Since there are a ton of images, just adding the path may slow down other lookups if you aren't using it. Compiling all the emojis on deploy will add overhead to your deploy if even the images haven't changed. Theres just so many more superfluous files to iterate over. Also, the urls will be fingerprinted which may not be ideal for referencing from cached content.


Example Rails Helper
--------------------

This would allow emojifying content such as: `it's raining :cat:s and :dog:s!`

See the [Emoji cheat sheet](http://www.emoji-cheat-sheet.com) for more examples.

```ruby
module EmojiHelper
  def emojify(content)
    h(content).to_str.gsub(/:([\w+-]+):/) do |match|
      if emoji = Emoji.find_by_alias($1)
        %(<img alt="#$1" src="#{image_path("emoji/#{emoji.image_filename}")}" style="vertical-align:middle" width="20" height="20" />)
      else
        match
      end
    end.html_safe if content.present?
  end
end
```

Unicode mapping
---------------

Translate emoji names to unicode and vice versa.

```ruby
>> Emoji.find_by_alias("cat").raw
=> "🐱"  # Don't see a cat? That's U+1F431.

>> Emoji.find_by_unicode("\u{1f431}").name
=> "cat"
```

Adding new emoji
----------------

You can add new emoji characters to the `Emoji.all` list:

```ruby
emoji = Emoji.create("music") do |char|
  char.add_alias "song"
  char.add_unicode_alias "\u{266b}"
  char.add_tag "notes"
end

emoji.name #=> "music"
emoji.raw  #=> "♫"
emoji.image_filename #=> "unicode/266b.png"

# Creating custom emoji (no Unicode aliases):
emoji = Emoji.create("music") do |char|
  char.add_tag "notes"
end

emoji.custom? #=> true
emoji.image_filename #=> "music.png"
```

As you create new emoji, you must ensure that you also create and put the images
they reference by their `image_filename` to your assets directory.

You can customize `image_filename` with:

```ruby
emoji = Emoji.create("music") do |char|
  char.image_filename = "subdirectory/my_emoji.gif"
end
```

For existing emojis, you can edit the list of aliases or add new tags in an edit block:

```ruby
emoji = Emoji.find_by_alias "musical_note"

Emoji.edit_emoji(emoji) do |char|
  char.add_alias "music"
  char.add_unicode_alias "\u{266b}"
  char.add_tag "notes"
end

Emoji.find_by_alias "music"       #=> emoji
Emoji.find_by_unicode "\u{266b}"  #=> emoji
```
