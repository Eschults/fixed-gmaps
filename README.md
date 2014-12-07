## Fixed map

Pour obtenir une page avec une carte fixe et du contenu que l'on peut faire défiler, comme sur [cette page](http://airbnbflo.herokuapp.com/flats), suivez les étapes ci-dessous.

Plaçons-nous (au hasard) dans le contexte d'une carte contenant des appartements à réserver.

### Step 1 - HTML

La vue correspondante doit contenir deux ```<div>``` :
- la carte, donnons-lui l'id **#map**;
- la liste des appartements, donnons-lui l'id **#sidebar**.

```html
# app/views/flats/index.html.erb

# la div correspondant à la carte
<div class="box-shadow" id="map"></div>

# la div correspondant à la liste des flats que nous voulons faire défiler
<div class="grey-bg" id="sidebar">
  # Il s'agit ici du code correspondant à l'exemple, mais l'important
  # est de placer ici votre contenu à faire défiler
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

#### La carte : **div#map**

Afin de fixer la carte, donnons-lui les propriétés css ci-dessous, à savoir :
- une ```position: absolute;```, ```top: 50px;``` et ```bottom: 0;``` pour qu'elle soit fixée et qu'elle occupe la totalité de la hauteur de la page, bien disposée sous la navbar qui a une ```height: 50px;```,
- ```left: 0;``` ```right: 40%;``` ```width: 40%;``` pour qu'elle soit calée **à gauche** et qu'elle prenne **40%** de la largeur du ```<body>```,
- ```z-index: 1;``` n'est pas indispensable, mais cette propriété permet de nous assurer que dans le cas où la "responsivité" du site n'est pas optimale et que des ```<div>``` seraient susceptibles de se superposer sur la carte, alors la carte resterait bien au premier plan.

#### Le contenu : **div#sidebar**

Pour que la sidebar soit correctement disposée, donnons-lui les propriétés complémentaires à celles de **#map**, soit une ```width: 60%;```, et ```right: 0;``` pour qu'elle soit bien calée **à droite** qu'elle occupe **60%** de la largeur du ```<body>```.
Ajoutons-lui maintenant la propriété ```overflow: auto``` qui permet d'ajouter une scrollbar à la div pour pouvoir faire défiler son contenu.

#### Le **```<body>```**

Enfin, nous devons empêcher la "scrollbar" du body d'apparaitre, ce qui nous assure que la carte restera bien de marbre en toutes circonstances.

Pour cela, nous lui ajoutons la propriété ```overflow: hidden```.
Les autres propriétés appliquées au ```<body>``` ont pour but d'enlever marges et padding, pour occuper la totalité de la page.

Le code est donc le suivant :

```css
#app/assets/stylesheets/map.css.scss

html, body {
  height: 100%;
  overflow: hidden;
  margin: 0;
  padding: 0;
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

A l'étape précédente, nous avons empêché la srollbar du ```<body>``` d'apparaitre, ce qui peut s'avérer légèrement handicapant pour les autres pages de notre site.

Nous devons donc nous assurer que cette propriété ne sera appliquée qu'à la vue correspondante.

C'est pourquoi nous avons créé un fichier à part dans ```app/assets/stylesheets```, que nous avons nommé ```map.css.scss``` pour y placer le code ci-dessus.

Ajoutons donc dans la ```<head>``` de notre ```app/views/layouts/application.html.erb``` le code suivant :

```html
# app/views/layouts/application.html.erb

<head>
  # reste de la head
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

### Sources

- [Stack overflow](http://stackoverflow.com/questions/15147378/position-google-maps-with-sidebar-on-right-and-fixed-header-on-top)
- [JS fiddle](http://jsfiddle.net/kuXYq/4/)

Enjoy :)