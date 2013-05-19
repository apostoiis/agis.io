---
layout: post
title: "Validating Paperclip image dimensions in Rails"
date: 2012-09-14 14:52
comments: true
categories: rails, paperclip
---

I love using [Paperclip](http://github.com/thoughtbot/paperclip) for handling image uploads in my Rails apps. It's super easy to use and [plays nice with Amazon S3](https://github.com/thoughtbot/paperclip#storage). It even has validators for attachment's content-type, filesize &amp; presence.

What it lacks is a built-in validator for image dimensions, but I've found out that it's not so hard to build one yourself. In my last project, [BathLiving](http://bathliving.herokuapp.com), I needed the user to upload images of specific dimensions. These images where going to be used in image sliders, thumbnails etc. I didn't want to use ImageMagick's resizing/scaling options since I wanted the quality of the image to remain intact. So I had to force the admins to create images with the proper width &amp; height in the first place.

Suppose this is our model:

{% highlight ruby %}
# app/models/product.rb

class Product < ActiveRecord::Base
  attr_accessible :title, :image

  has_attached_file :image, styles: { thumb: '298x120#' }

  validates_attachment :image, presence: true

  validate :image_dimensions

  private

  def image_dimensions
    required_width  = 200
    required_height = 200
    dimensions = Paperclip::Geometry.from_file(image.queued_for_write[:original].path)

    errors.add(:image, "Width must be #{width}px") unless dimensions.width == required_width
    errors.add(:image, "Height must be #{height}px") unless dimensions.height == required_height
  end
end
{% endhighlight %}

I believe the code speaks for itself here. Pretty simple, right? We use [#from_file](http://rdoc.info/github/thoughtbot/paperclip/Paperclip/Geometry#from_file-class_method) that Paperclip::Geometry gives us to retrieve the dimensions of the image that the user is *trying* to upload, in other words, the image that's queued for write.

But there's one major drawback in this implementation: *it's not reusable*. What if we need to validate the dimensions in other models too? We'd have to do the same thing over and over in our models.

[Custom validators](http://guides.rubyonrails.org/active_record_validations_callbacks.html#custom-validators) to the rescue. It looks like this:

{% highlight ruby %}
# app/models/dimensions_validator.rb

class DimensionsValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    if record.send("#{attribute}?".to_sym)
      dimensions = Paperclip::Geometry.from_file(value.queued_for_write[:original].path)
      width = options[:width]
      height = options[:height]

      record.errors[attribute] << "Width must be #{width}px" unless dimensions.width == width
      record.errors[attribute] << "Height must be #{height}px" unless dimensions.height == height
    end
  end
end
{% endhighlight %}

First we check that the attachment is actually present (line 5). If it is, we perform the validations.

Now in our models we can simply do this:

{% highlight ruby %}
validates :image, dimensions: { width: 200, height: 200 }
{% endhighlight %}

and Rails will automatically find our custom validator since it's in the right place (app/models/) with the right name (dimensions_validator).

Now this implementation won't work with ActiveRecord associations (eg. if the attachments are declared on a separate Image model), but for the common cases where the attachments and their parent records are in the same model it's all good.
