## Fixed map

Pour obtenir une page avec une carte fixe et du contenu que l'on peut faire défiler, comme sur [cette page](http://airbnbflo.herokuapp.com/flats), suivez les étapes ci-dessous.

Plaçons-nous (au hasard) dans le contexte d'une carte contenant des appartements à réserver.

### Step 1 - HTML

La vue correspondante doit contenir deux ```<div>``` :
- la carte, donnons-lui l'id **#map**;
- la liste des appartements, donnons-lui l'id **#sidebar**.

```erb
<!-- app/views/flats/index.html.erb -->

<!-- div correspondant à la carte -->
<div class="box-shadow" id="map"></div>

<!-- div avec liste déroulante d'appartements -->
<div class="grey-bg" id="sidebar">
  <h1 class="padding-left-15">List of flats</h1>
  <% @flats.each do |f| %>
    <div class="col-sm-12 col-md-6 index-flats">
      <h3><%= link_to f.title, flat_path(f) %></h3>
      <div class="picture-placeholder">
        <% if f.photos.first %>
          <%= image_tag f.photos.first.picture.url(:medium), class: "img-thumbnail"%>
        <% else %>
          <%= image_tag "images.png", class: "img-thumbnail flat-picture" %>
        <% end %>
      </div>
    </div>
  <% end %>
</div>

# ici le code js pour afficher la carte et les marqueurs
```

### Step 2 - CSS

#### La carte : **`div#map`**

Afin de fixer la carte, donnons-lui les propriétés css ci-dessous afin qu'elle occupe toute la hauteur de la page, qu'elle soit calée à gauche et occupe 40% de la largeur de la page.

```css
position: absolute;
top: 50px; /* assuming a navbar with a 50px height */
bottom: 0;
left: 0;
right: 40%;
```


Vous pouvez ajouter un `z-index: 1;`. Ce n'est pas indispensable mais cette propriété permet de nous assurer que dans le cas où la "responsivité" du site n'est pas optimale et que des `<div>` seraient susceptibles de se superposer sur la carte, alors la carte resterait bien au premier plan.

#### Le contenu : **`div#sidebar`**

Pour que la sidebar soit correctement disposée, donnons-lui les propriétés complémentaires à celles de **`#map`**,

```css
position: absolute;
width: 60%;
top: 50px;
right: 0;
bottom: 0;
overflow: auto; /* permet de rendre le contenu scrollable */
```

#### Le **`<body>`**

Enfin, nous devons empêcher la "scrollbar" du body d'apparaitre, ce qui nous assure que la carte restera bien de marbre en toutes circonstances. Pour cela, nous lui ajoutons la propriété ```overflow: hidden```.

Le code est donc le suivant :

```css
/* app/assets/stylesheets/map.css.scss */

html, body {
  overflow: hidden;
}

#map {
  position: absolute;
  top: 50px;
  right: 40%;
  width: 40%;
  bottom: 0;
  left: 0;
  z-index: 1;
}

#sidebar {
  position: absolute;
  width: 60%;
  top: 50px;
  right: 0;
  bottom: 0;
  overflow: auto;
  padding: 15px;
}
```

### Step 3 - Organisation des fichiers

#### Body scrollbar

A l'étape précédente, nous avons empêché la scrollbar du `<body>` d'apparaitre, ce qui peut s'avérer légèrement handicapant pour les autres pages de notre site.

Nous devons donc nous assurer que cette propriété ne sera appliquée qu'à la vue correspondante.

C'est pourquoi nous avons créé un fichier à part dans ```app/assets/stylesheets```, que nous avons nommé ```map.css.scss``` pour y placer le code ci-dessus.

Ajoutons donc dans la ```<head>``` de notre ```app/views/layouts/application.html.erb``` le code suivant :

```erb
<!-- app/views/layouts/application.html.erb -->

<head>
  <!-- reste du head -->
  <%= yield :map_css %>
</head>
```

Et dans notre vue ```app/views/flats/index.html.erb``` le code suivant :

```html
# app/views/flats/index.html.erb

<% content_for :map_css do %>
  <%= stylesheet_link_tag 'map' %>
<% end %>

# reste de votre code

```

Ainsi, les propriétés du fichier ```map.css.scss``` ne seront appliquées qu'à cette vue.

Mais pour cela, comme nous n'avons pas fait d'`@import "map";` dans notre `application.css.scss` (toujours pour que ses propriétés ne soient pas appliquées à toutes les pages de notre site), nous devons faire en sorte que `map.css.scss` soit chargée au lancement de l'application.

Rendez-vous donc dans votre fichier ```config/initializers/assets.rb``` et ajoutez-y :

```ruby
#config/initializers/assets.rb

# reste du code
Rails.application.config.assets.precompile += %w( map.css )
```

#### Footer

Enfin, si vous avez placé votre footer dans votre layout générale (ce qui est bien sûr la bonne pratique), vous pourriez vouloir vous en affranchir sur cette page spécifique, car il risque d'apparaitre un peu n'importe où maintenant que nous avons donné des ```position: absolute;``` à nos autres ```<div>```.

Pour cela, nous allons créer une layout spécifique que nous allons appeler ```app/views/layouts/map.html.erb```.

Cette layout n'est rien d'autre que notre layout générale (faites donc un bon copier-coller des familles de votre layout application vers la nouvelle layout map), à laquelle nous retirons la ligne correspondant à l'appel du footer soit ```<%= render "layouts/footer" %>```. Nous pouvons aussi en profiter pour retirer de la layout générale le ```<% yield :map_css %>``` qui sera désormais inutile à cet endroit de notre code.

Maintenant, nous souhaitons dire à notre controller de flats de ne plus appeler la layout ```app/views/layouts/application.html.erb```, mais la layout ```app/views/layouts/map.html.erb``` quand son action **index** est appelée.

Pour cela, rendez-vous dans votre controller ```app/controllers/flats_controller.rb```, et ajoutez-y le code suivant :

```ruby
# app/controllers/flats_controller.rb

class FlatsController < ApplicationController
  # vos before_action ou skip_before_action...
  layout 'map', only: [:index]

  # reste de votre code
end
```

#### Pour aller plus loin

Maintenant que nous appelons une layout spécifique pour notre page d'index des flats, nous pouvons :

```html
# app/views/layouts/map.html.erb

<head>
  # reste de la head

  # Retirer la ligne ci-dessous
  <%= yield :map_css %>

  # La remplacer par :
  <%= stylesheet_link_tag 'map' %>
</head>
```


```html
# app/views/flats/index.html.erb

# Retirer les 3 lignes ci-dessous :
<% content_for :map_css do %>
  <%= stylesheet_link_tag 'map' %>
<% end %>

# reste de votre code
```

### Sources

- [Stack overflow](http://stackoverflow.com/questions/15147378/position-google-maps-with-sidebar-on-right-and-fixed-header-on-top)
- [JS fiddle](http://jsfiddle.net/kuXYq/4/)

Enjoy :)
