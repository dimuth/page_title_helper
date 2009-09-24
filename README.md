# Page title helper

Ever wondered if there was an easier and DRY-way to set your page titles (and/or headings). Backed
by Rails (only tested on 2.3.x) and it's new <tt>I18n</tt>-class the solution is a simple helper method.

In your layout add this to your <tt><head>-section</tt>:

      ...
      <title><%=h page_title %></title>
      ...
      
That's it. Now just add your translations, to the locales, in e.g. <tt>config/locales/en.yml</tt>:

      en:
        contacts:
          index:
            title: "Contacts"
            
When <tt>contacs/index.html.erb</tt> is rendered, the key <tt>:en, :contacts, :index, :title</tt>
is looked up and printed, together with the applications basename, like: <tt>My cool app - Contacts</tt>.
The format etc. is of course configurable, just head down to the options.

## Customize titles

Need a custom title, or need to fill in some placeholders? Just provide a block, in e.g.
<tt>contacts/show.html.erb</tt> the requirement is to display the contacts name in the
<tt><title>-tag</tt> as well as in the heading?

    <h1><%=h page_title { @contact.name } %></h1>
    
A call to <tt>page_title</tt> will now return the contacts name, neat :) if for example the
<tt>h1</tt> does not match the +title+, then
well, just do something like:

    <% page_title { @contact.name + " (" + @contact.company.name + ")" } %>
    <h1><%=h @contact.name %></h1>
    
Guess, that's it. Of course it's also possible to use +translate+ within the +page_title+ block, so
to translate customzied titles, like:

    # in config/locales/en.yml:
    en:
      dashboard:
        index:
          title: "Welcome back, {{name}}"

    # in app/views/dashboard/index.html.erb:
    <h1><%=h page_title { t '.title', :name => @user.first_name } %></h1>
    
Btw - a helpful rule-of-thumb: if +page_title+ is used with a +block+ a title is <b>defined</b>,
if it's used without the current title is rendered.

## More fun with <tt>:format</tt>

The <tt>:format</tt> option is used to specify how a title is formatted, i.e. if the app name is
prependor appended, or if it contains the account name etc. It uses a similar approach as
<tt>paperclip</tt>s path interpolations:

    page_title :format => ':title / :app' # => "Contacts / My cool app"
    
Adding custom interpolations is as easy as defining a block, for example to access the current
controller:

    PageTitleHelper.interpolates :controller do |env|
      env.controller.controller_name.humanize
    end
    
    page_title :format => ':title / :controller / :app' # => "Welcome back / Dashboard / My cool app"
    
To access just the title, without any magic app stuff interpolated or appended, use:

    page_title { "untitled" }
    page_title :format => false # => "untitled"

## All options - explained

<table>
  <tr>
    <th>Option</th><th>Description</th><th>Default</th><th>Values</th>
  </tr>
  <tr>
    <td><tt>:app</tt></td>
    <td>Specify the applications name, however it's
        recommended to define the translation key <tt>:'app.name'</tt>.</td>
    <td>Inflected from <tt>RAILS_ROOT</tt></td>
    <td>string</td>
  </tr>
  <tr>
    <td><tt>:default</tt></td>
    <td>String which is displayed when no translation exists and no custom title
        has been specified. Can also be set to a symbol or array to take advantage of
        <tt>I18n.translate</tt>s <tt>:default</tt> option.</td>
    <td><tt>:'app.tagline'</tt></td>
    <td>string, symbol or array of those</td>
  </tr>
  <tr>
    <td><tt>:format</tt></td>
    <td>Defines the output format, accepts a string containing multiple interpolations,
        see <i>More fun with <tt>:format</tt></i>. If set to +false+, just the current title is returned.
        If <tt>:format => :app</tt> then just the application name is returned.</td>
    <td><tt>":app - :title"</tt></td>
    <td>Any string, containg "formatting" patterns</td>
  </tr>
  <tr>
    <td><tt>:suffix</tt></td>
    <td>Not happy with the fact that the translations must be named like
        <tt>en -> contacts -> index -> title</tt>, but prefer e.g. them to be suffixed with
        <tt>page_title</tt>? Then just set <tt>:suffix => :page_title</tt>.</td>
    <td><tt>:title</tt></td>
    <td>symbol or string</td>
  </tr>
</table>
</p>

If an option should be set globally it's possible to change the default options hash as follows:

    PageTitleHelper.options[:format] = ':title / :app'
    
Note, currently it only makes sense to set <tt>:format</tt> and/or <tt>:default</tt> globally.

## Some (maybe useful) interpolations

The internationalized controller name, with fallback to just display the humanized name:

    PageTitleHelper.interpolates :controller do |env|
      I18n.t env.controller.controller_path.tr('/','.') + '.controller', :default => env.controller.controller_name.humanize
    end
    
<b>Note:</b> Where should I put these, as well as the default options? I'd put them in a new file at <tt>config/initializers/page_title.rb</tt> or someting like that.
    
## Licence and copyright
Copyright (c) 2009 Lukas Westermann (Zurich, Switzerland), released under the MIT license