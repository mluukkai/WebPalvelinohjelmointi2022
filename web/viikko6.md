Jatkamme sovelluksen rakentamista siitä, mihin jäimme viikon 5 lopussa. Allaoleva materiaali olettaa, että olet tehnyt kaikki edellisen viikon tehtävät. Jos et tehnyt kaikkia tehtäviä, voit täydentää ratkaisusi tehtävien palautusjärjestelmän kautta näkyvän esimerkivastauksen avulla.

## Testeistä

Osa tämän viikon tehtävistä saattaa hajottaa jotain edellisinä viikkoina tehtyjä testejä. Voit merkitä tehtävät testien hajoamisesta huolimatta, eli testien pitäminen kunnossa on vapaaehtoista.

## Muistutus debuggerista

Viikolla 2 tutustuimme [debuggeriin](https://github.com/mluukkai/WebPalvelinohjelmointi2018/blob/master/web/viikko2.md#debuggeri). Valitettavasti debuggeri ei ole vielä löytänyt tietänsä jokaisen kurssilaisen työkaluvalikoimaan.

Debuggerin käyttö on erittäin helppoa. Riittää kirjoittaa komento <code>binding.pry</code> (tai <code>byebug</code>)  _mihin tahansa_ kohtaan sovelluksen koodia. Seuraavassa esimerkki:

```ruby
class PlacesController < ApplicationController
   # ...

  def search
    city = params[:city]
    binding.pry
    @places = BeermappingApi.places_in(city)
    if @places.empty?
      redirect_to places_path, notice: "No locations in #{city}"
    else
      @weather = ApixuApi.weather_in(city)
      session[:city] = city
      render :index
    end
  end
```

Tarkastelemme siis debuggerilla BeermappingApia käyttävää osaa sovelluksesta. Kun nyt sovelluksella haetaan jotain olutravintolaa avaa debuggeri konsolisession koodiin merkittyyn kohtaan:

```ruby
From: /Users/mluukkai/opetus/ratebeer/app/controllers/places_controller.rb @ line 10 PlacesController#search:

     9: def search
 => 10:   binding.pry
    11:   @places = BeermappingApi.places_in(params[:city])
    12:   if @places.empty?
    13:     redirect_to places_path, notice: "No locations in #{params[:city]}"
    14:   else
    15:     @weather = ApixuApi.weather_in(params[:city])
    16:     session[:city] = params[:city]
    17:     render :index
    18:   end
    19: end

>  params
=> <ActionController::Parameters {"utf8"=>"✓", "authenticity_token"=>"b6MZBx2+UL1dc8uWnCxmjNiCGHtODDMf6PgtB6OQAjxQL15jAZ2NO+YyHOVIFO/96qVJAYJktBKPB4hUeWJEHA==", "city"=>"turku", "commit"=>"Search", "controller"=>"places", "action"=>"search"} permitted: false>
```

eli pystymme mm. tarkastamaan että <code>params</code> hashin sisältö on sellainen kuin oletamme sen olevan.

Suoritetaan sitten seuraava komento ja katsotaan että tulos on odotetun kaltainen. Jos käytämme _byebugia_ voi seuraavan komennon suorittaa komennolla <code>next</code>. Pry ei tätä mahdollisuutta tarjoa ja seuraava komento pitää käytännössa suoritta kopioimalla komento terminaaliin:

```ruby
> @places = BeermappingApi.places_in(city)
=> @places = BeermappingApi.places_in(city)
=> [#<Place id="18856", name="Panimoravintola Koulu", status="Brewpub", reviewlink="https://beermapping.com/location/18856", proxylink="http://beermapping.com/maps/proxymaps.php?locid=18856&d=5", blogmap="http://beermapping.com/maps/blogproxy.php?locid=18856&d=1&type=norm", street="Eerikinkatu 18", city="Turku", state=nil, zip="20100", country="Finland", phone="(02) 274 5757", overall="0", imagecount="0">]
> @places.size
=> 1
>@places.first.name
"Panimoravintola Koulu"
> exit
```

viimeinen komento jatkaa ohjelman normaalia suorittamista.

Debuggerin voi siis käynnistää _mistä tahansa kohtaa_ sovelluksen koodia, myös testeistä tai jopa näkymistä. Kokeillaan debuggerin käynnistämistä uuden oluen luomislomakkeen renderöinnin aikana:

```erb
     9:       <% end %>
    10:       </ul>
    11:     </div>
    12:   <% end %>
    13:
 => 14:   <% binding.pry %>
    15:
    16:   <div class="field">
    17:     <%= f.label :name %><br>
    18:     <%= f.text_field :name %>
    19:   </div>

> @styles.size
=> 7
> @styles.first
=> #<Style:0x00007f8edc0ca4f0
 id: 1,
 name: "European pale lager",
 description:
  "Similar to Munich Helles, many European countries reacted to the popularity of early pale lagers by brewing their own. Hop flavor is significant and of noble varieties, bitterness is moderate, and both are backed by a solid malt body and sweet notes from an all-malt base.",
 created_at: Thu, 20 Sep 2018 10:17:39 UTC +00:00,
 updated_at: Thu, 20 Sep 2018 10:35:04 UTC +00:00>
````

Näkymätemplateen on siis lisätty <code><% binding.pry %></code>. Kuten huomaamme, on jopa näkymän apumetodin <code>options_from_collection_for_select</code> kutsuminen mahdollista debuggerista käsin:

```ruby
> options_from_collection_for_select(@styles, :id, :name, selected: @beer.style_id)
  Style Load (0.2ms)  SELECT "styles".* FROM "styles"
=> "<option value=\"1\">European pale lager</option>\n<option value=\"2\">Pale Ale</option>\n<option value=\"3\">Porter</option>\n<option value=\"4\">German hefeweizen</option>\n<option value=\"5\">IPA</option>\n<option value=\"6\">Ilowalcohol</option>\n<option value=\"7\">Pale ale</option>"
```

Eli vielä kertauksena **kun kohtaat ongelman, turvaudu arvailun sijaan byebugiin!**

Rails-konsolin käytön tärkeyttä sovelluskehityksen välineenä on yritetty korostaa läpi kurssin. Eli **kun teet jotain vähänkin epätriviaalia, testaa asia ensin konsolissa.** Joissain tilanteissa voi olla jopa parempi tehdä kokeilut debuggerin avulla avautuvassa konsolissa, sillä tällöin on mahdollista avata konsolisessio juuri siihen kontekstiin, mihin koodia ollaan kirjoittamassa. Näin ollen päästään käsiksi esim. muuttujiin <code>params</code>, <code>sessions</code> ym. suorituskontekstista riippuvaan dataan.

## Bootstrap

Toistaiseksi emme ole kiinnittäneet ollenkaan huomiota sovelluksiemme ulkoasuun. Modernin ajattelun mukaan HTML-koodi määrittelee ainoastaan sivujen tietosisällön ja ulkoasu määritellään erillisissä CSS-tiedostoissa.

HTML:ssä merkataan elementtejä _luokilla_ (class), ja _id_:illä, jotta tyylitiedostojen määrittelemiä tyylejä saadaan ohjattua halutuille kohtiin sivua.

Määrittelimme jo muutama viikko sitten, että application layoutiin sijoittamamme navigointipalkki sijaitsee div-elementisssä jolle on asetettu luokka "navibar":

```erb
<div class="navibar">
  <%= link_to 'breweries', breweries_path %>
  <%= link_to 'beers', beers_path %>
  <%= link_to 'styles', styles_path %>
  <%= link_to 'ratings', ratings_path %>
  <%= link_to 'users', users_path %>
  <%= link_to 'clubs', beer_clubs_path %>
  <%= link_to 'places', places_path %>
  |
  <% if not current_user.nil? %>
    <%= link_to current_user.username, current_user %>
    <%= link_to 'rate a beer', new_rating_path %>
    <%= link_to 'join a club', new_membership_path %>
    <%= link_to 'signout', signout_path, method: :delete %>
  <% else %>
    <%= link_to 'signin', signin_path %>
    <%= link_to 'signup', signup_path %>
  <% end %>
</div>
```

Määrittelimme viikolla 2 navigointipalkille tyylin lisäämällä hakemistossa app/assets/stylesheets/ sijaitsevaan tiedostoon application.css seuraavat:

```css
.navibar {
  padding: 10px;
  background: #EFEFEF;
}
```

CSS:ää käyttämällä koko sivuston ulkoasu voitaisiin muotoilla sivuston suunnittelijan haluamalla tavalla, jos silmää ja kykyä muotoiluun löytyy.

Sivuston muotoilunkaan suhteen ei onneksi ole enää tarvetta keksiä pyörää uudelleen. Bootstrap http://getbootstrap.com/ on "kehys", joka sisältää suuren määrän web-sivujen ulkoasun muotoiluun tarkoitettuja CSS-tyylitiedostoja ja javascriptiä. Bootstrap onkin noussut nopeasti suureen suosioon web-sivujen ulkoasun muotoilussa.

Aloitetaan sitten sovelluksemme bootstrappaaminen gemin <https://github.com/twbs/bootstrap-rubygem> avulla. Lisätään Gemfileen seuraavat:

```ruby
gem 'bootstrap', '~> 4.1.3'
gem 'jquery-rails', '>= 4.3.3'
```

Asennetaan gemit komennolla <code>bundle install</code>, asennuksen jälkeen sovellus tulee uudelleenkäynnistää.

Gemin [asennusohjetta](https://github.com/twbs/bootstrap-rubygem#a-ruby-on-rails) noudattaen lisätään tiedoston _app/assets/javascript/application.js_ loppuun seuraavat 

```
//= require jquery3
//= require popper
//= require bootstrap-sprockets
```

Muutetaan myös tiedoston _app/assets/stylesheets/application.css_ pääte muotoon _scss_ ja lisätään tiedoston loppuun rivi

```
@import "bootstrap";
```

Kun nyt avaamme sovelluksen selaimella (ja sovellus on uudelleenkäynnistettu), huomaamme jo pienen muutoksen esim. fonteissa. 

### Navbar

Bootstrapissa käyttöliittymä rakennetaan CSS-luokkina määritellyistä komponenteista. Eräs esimerkki bootstrapin komponenteista on [navbar](https://getbootstrap.com/docs/4.0/components/navbar/), jonka avulla voidaan muotoilla sovelluksen navigaatiopalkki.

Muutetaan tiedosto _app/views/layouts/application.html.erb_  seuraavaan muotoon:

```ruby
<!DOCTYPE html>
<html>
  <head>
    <title>Ratebeer</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>

    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
      <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent">
        <span class="navbar-toggler-icon"></span>
      </button>

      <div class="collapse navbar-collapse" id="navbarSupportedContent">
        <ul class="navbar-nav mr-auto">
          <li class="nav-item">
            <%= link_to 'breweries', breweries_path, { class: "nav-link" } %>
          </li>
          <li class="nav-item">
            <%= link_to 'beers', beers_path, { class: "nav-link" } %>
          </li>
          <li class="nav-item">
            <%= link_to 'styles', styles_path, { class: "nav-link" } %>
          </li>
          <li class="nav-item">
            <%= link_to 'ratings', ratings_path, { class: "nav-link" } %>
          </li>
          <li class="nav-item">
            <%= link_to 'users', users_path, { class: "nav-link" } %>
          </li>
          <li class="nav-item">
            <%= link_to 'clubs', beer_clubs_path, { class: "nav-link" } %>
          </li>
          <li class="nav-item">
            <%= link_to 'places', places_path, { class: "nav-link" } %>
          </li>
          <% if current_user %>
            <li class="nav-item">
              <%= link_to current_user.username, current_user, { class: "nav-link" } %>
            </li>
            <li class="nav-item">
              <%= link_to 'rate a beer ', new_rating_path, { class: "nav-link" } %>
            </li>
            <li class="nav-item">
              <%= link_to 'join a club ', new_membership_path, { class: "nav-link" } %>
            </li>
            <li class="nav-item">
              <%= link_to 'signout', signout_path, { class: "nav-link", method: :delete  } %>
            </li>
          <% else %>
            <li class="nav-item">
              <%= link_to 'signin', signin_path, { class: "nav-link" } %>
            </li>
            <li class="nav-item">
              <%= link_to 'signup', signup_path, { class: "nav-link" } %>
            </li>
          <% end %>                            
        </ul>
      </div>
    </nav>

    <%= yield %>
  </body>
</html>
```

Bootstrapin dokumentaatio ei ole ihan selkein mahdollinen, mutta pienellä ihmettelyllä saimme aikaan navigaatiopalkin, joka on sisällöltään samanlainen entisen kaltainen.

Vaikka bootstrapilla muotoiltu navigaatiopalkki on koodina pidempi ja sotkuisempikin kuin aiempi navigaatiopalkkimme, on sillä kuitenkin eräs merkittävä etu. Jos sovellusta tarkastellaan "isolta" näytöltä, näkyy navigaatiopalkki normaalisti:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w6-0a.png)

Jos taas sovellusta tarkastellaan pienemmältä näytöltä, esim. mobiililaitteelta, näytetään navigaatiopalkin sijaan symboli, jota klikkaamalla navigaatiopalkki aukeaa alaspäin:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w6-0b.png)

Bootstrapilla muotoiltu navigaatiopalkki on _responsiivinen_, se mukautuu sovellusta käyttävän selaimen kokoon.

### Grid

Sen lisäksi että Bootstrapilla voi helposti muodostaa responsiivisen navigointipalkin, voidaan Bootstrapin grid-järjestelmän avulla jakaa sivu erillisiin osiin, ks. https://getbootstrap.com/docs/4.0/layout/grid/

Muutetaan tiedoston _app/views/layout/application.html.erb_ alaosassa oleva yksittäisten näkymätemplatejen renderöinnin kohdan merkkaava 

```erb
<%= yield %>
```     

muotoon

```erb
<div class="container">
  <div class="row">
    <div class="col-sm-8">
      <%= yield %>
    </div>
    <div class="col-sm-4">
      <img
        src="http://www.cs.helsinki.fi/u/mluukkai/wadror/pint.jpg"
        width="200"
        style="padding-top:30px"
      >
    </div>
  </div>
</div>
```

Eli sijoitamme bootstrapin containeriin, eli varsinaisen sivun sisällön sisältävään osaan yhden rivin, jonka jaamme kahteen sarakkeeseen: 8:n levyiseen johon kunkin sivun tiedot upotetaan ja 4:n levyiseen osaan jossa näytämme kuvan riippumatta siitä millä sivulla ollan.

Sivun pohja on nyt kunnossa ja voimme hyödyntää bootstrapin tyylejä ja komponentteja sivuillamme.

### notifikaatio

Useissa sovelluksen näkymissä on rivi

```erb
<p id="notice"><%= notice %></p>
```

jonka avulla käyttäjälle näytetään erilaisia notifikaatioita, mm. _Beer was successfully created._

Notifikaatiot kannattaa muotoilla bootstrapin [alert](https://getbootstrap.com/docs/4.0/components/alerts/)-komponentin avulla:


```erb
<% if notice %>
  <div class="alert alert-primary" role="alert">
    <%= notice %>
  </div>
<% end %>
```


Sen sijaan että tekisimme lisäisimme muutoksen jokaiselle sivulle, millä notifikaation näyttävä koodi on, on parempi siirtää notifikaation näyttävä logiikka   tiedostoon _app/views/layout/application.html.erb_

```erb
<div class="container">
  <% if notice %>
    <div class="alert alert-primary" role="alert">
      <%= notice %>
    </div>
  <% end %>

  <div class="row">
    ...
  </div>
</div>
```

ja poistaa se muista näkymätiedostoista, kuten _app/views/beers/index.html.erb_

Jos käytät Visual Studio Codea, niin voit käyttää _replace in files_ -toimintoa poistamaan ylimääräiseksi jääneet `<p id="notice"><%= notice %></p>` -komennot.

### lisää komponentteja

Muotoillaan seuraavaksi hieman sivulla käyttämiämme taulukoita. Bootstrapin sivulta https://getbootstrap.com/docs/4.0/content/tables/ näemme, että taulukon normaali bootstrap-muotoilu saadaan käyttöön lisäämällä taulukon HTML-koodille luokka <code>table</code>, seuraavasti:

```erb
<table class="table">
  ...
</table>
```

Lisätään luokkamäärittely esim. oluiden sivulle ja kokeillaan. Näyttää jo paljon professionaalimmalta. Päätetään vielä lisätä luokka <code>table-hover</code>, jonka ansioista se rivi jonka kohdalla hiiri on muuttuu korostetuksi, eli taulukon luokkamäärittelyksi tulee

```erb
<table class="table table-hover">
  ...
</table>
```

> ## Tehtävä 1
>
> Muuta ainakin muutama sovelluksen taulukoista käyttämään bootstrapin tyylejä.
>
> VS Coden käyttäjille muutos onnistuu helposti _replace in files_ -toiminnolla

Bootstrap tarjoaa valmiit tyylit myös painikkeille https://getbootstrap.com/docs/4.0/components/buttons/

Päätetään käyttää luokkaparin <code>btn btn-primary</code> määrittelemää sinistä painiketta. Seuraavassa esimerkki, missä luokka on lisätty oluen reittauksen tekevälle painikkeelle:

```erb
  <h4>give a rating:</h4>

  <%= form_for(@rating) do |f| %>
    <%= f.hidden_field :beer_id %>
    score: <%= f.number_field :score %>
    <%= f.submit class:"btn btn-primary" %>
  <% end %>
```

Luokka voidaan lisätä myös niihin linkkeihin, jotka halutaan napin painikkeen näköisiksi:

```erb
<%= link_to('New Beer', new_beer_path, class:'btn btn-primary') if current_user %>
```

> ## Tehtävä 2
>
> Lisää sovelluksen ainakin muutamille painikkeille ja painikkeen tapaan toimiville linkeille valitut tyylit. Poisto-operaatioissa tyyliksi kannattaa laittaa <code>btn btn-danger</code>.

> ## Tehtävä 3
>
> Sovelluksemme lomakkeet ovat tällä hetkellä melko rumia. Tee ainakin uuden olutseuran luomislomakkeesta tyylikkäämpi Bootstrapin [lomakkeiden](https://getbootstrap.com/docs/4.0/components/forms/) muotoiluun tarkoitettujen komponenttien avulla. 
>
>Saat päättää lomakkeen tarkan tyylin itse. Eräs tapa muotoilla lomake on seuraava

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w6-3a.png)

> ## Tehtävä 4
>
> Muuta navigointipalkkia siten, että käyttäjän kirjautuessa kirjautunutta käyttäjää koskevat toiminnot tulevat menupalkin dropdowniksi alla olevan kuvan tapaan.
>
> Apua löydät [navbarin](https://getbootstrap.com/docs/4.0/components/navbar/) ohjeiden _dropdown_-elementtejä sisältävistä esimerkeistä.
>
> Ratkaisu ei ole kaikilta osin ihan suoraviivainen. Eräs mahdollisuus muokata apufunktion <code>link_to</code> tekemän linkin _class_ halutun kaltaiseksi:
>
> ```
> <%= link_to 'signout', signout_path, { class: "dropdown-item", method: :delete } %>
> ```

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w6-3c.png)

> ## Tehtävä 5
>
> Tee jostain sivustosi osasta tyylikkäämpi käyttämällä jotain Bootstrapin komponenttia. Saat merkitä rastin jos käytät aikaa sivustosi ulkoasun parantamiseen vähintään 15 minuuttia. 

## Panimon aktiivisuus

Osa panimoista on jo lopettanut toimintansa ja haluaisimme eriyttää lopettaneet panimot aktiivisten panimoiden listalta. Lisätään painimotietokantaan aktiivisuuden merkkaava boolean-arvoinen sarake. Luodaan migraatio:

    rails g migration AddActivityToBrewery active:boolean

Huom: koska migraation nimi alkaa sanalla Add ja loppuu olion nimeen Brewery, ja sisältää tiedon lisättävästä sarakkeesta, generoituu juuri oikea migraatiokoodi automaattisesti. Asia on kuitenkin syytä varmistaa tarkistamalla generoituneen migraatiotiedoston sisältö

```ruby
class AddActivityToBrewery < ActiveRecord::Migration[5.2]
  def change
    add_column :breweries, :active, :boolean
  end
end
```

Suoritetaan migraatio ja käydään konsolista käsin merkkaamassa kaikki tietokannassa olevat panimot aktiiviseksi:

```ruby
> Brewery.all.each{ |b| b.active=true; b.save }
```

Käydään luomassa uusi panimo, jotta saamme tietokantaamme myös yhden epäaktiivisen panimon.

Muutetaan sitten panimon sivua siten, että se kertoo panimon mahdollisen epäaktiivisuuden panimon nimen vieressä:

```erb
 <h2>
  <%= @brewery.name %>
  <% if not @brewery.active  %>
    <span class="badge badge-info">retired</span>
  <% end %>
</h2>

<p><em>Established at <%= @brewery.year %></em></p>

<p>Number of beers <%= @brewery.beers.count %> </p>

<p>
 <% @brewery.beers.each do |beer| %>
   <%= link_to beer.name, beer %>
 <% end %>
</p>

<p>
  <% if @brewery.ratings.empty? %>
    beer has not yet been rated!
  <% else %>
    Has <%= pluralize(@brewery.ratings.count, 'rating') %>, average <%= @brewery.average_rating %>
  <% end %>
</p>

<% if current_user %>
  <%= link_to 'Edit', edit_brewery_path(@brewery), class:"btn btn-primary" %>
  <%= link_to 'Destroy', @brewery, method: :delete, data: { confirm: 'Are you sure?' }, class:"btn btn-danger" %>
<% end %>

```

Panimon luomis- ja editointilomakkeeseen on syytä lisätä mahdollisuus panimon aktiivisuuden asettamiseen. Lisätään views/breweries/_form.html.erb:iin checkbox aktiivisuuden säätelyä varten:

```erb
<div class="field">
  <%= f.label :active %>
  <%= f.check_box :active %>
</div>
```

Kokeillaan. Huomaamme kuitenkin että aktiivisuuden muuttaminen ei toimi.

Syynä tälle on se, että attribuuttia <code>active</code> ei ole lueteltu  massasijoitettavaksi sallittujen attribuuttien joukossa.

Tutkitaan hieman panimokontrolleria. Sekä uuden panimon luominen, että panimon tietojen muuttaminen hakevat panimoon liittyvät tiedot metodin <code>brewery_params</code> avulla:

```ruby
def create
  @brewery = Brewery.new(brewery_params)

  # ...
end

def update
  # ...
  if @brewery.update(brewery_params)
  # ...
end

def brewery_params
  params.require(:brewery).permit(:name, :year)
end
```

Kuten [viikolla 2 totesimme](
https://github.com/mluukkai/WebPalvelinohjelmointi2018/blob/master/web/viikko2.md#reittauksen-talletus) on jokainen massasijoitettavaksi tarkoitettu attribuutti eksplisiittisesti sallittava <code>permit</code> metodin avulla. Muutetaan metodia <code>brewery_params</code> seuraavasti:
```ruby
def brewery_params
  params.require(:brewery).permit(:name, :year, :active)
end
```

Päätetään, että haluamme näyttää panimoiden listalla erikseen aktiiviset ja epäaktiiviset panimot. Suoraviivainen ratkaisu on seuraava. Talletetaan kontrollerissa aktiiviset ja passiiviset omiin muuttujiinsa:

```ruby
def index
  @active_breweries = Brewery.where(active: true)
  @retired_breweries = Brewery.where(active: [nil, false])
end
```

Kentän <code>active</code>-arvo voi olla joko eksplisiittisesti asetettu <code>false</code> tai <code>nil</code> jotka molemmat tarkoittavat eläköitynyttä panimoa, olemme joutuneet lisäämään jälkimmäiseen <code>where</code>-lauseeseen molemmat vaihtoehdot.

Copypastetaan näkymään taulukko kahteen kertaan, erikseen aktiivisille ja eläköityneille:

```erb
<h1>Breweries</h1>

<h2>Active</h2>

<p> Number of active breweries: <%= @active_breweries.count %> </p>

<table class="table table-hover">
  <thead>
    <tr>
    <th>Name</th>
    <th>Year</th>
    </tr>
  </thead>

  <tbody>
    <% @active_breweries.each do |brewery| %>
      <tr>
        <td><%= link_to brewery.name, brewery %></td>
        <td><%= brewery.year %></td>
        <td></td>
      </tr>
    <% end %>
  </tbody>
</table>

<h2>Retired</h2>

<p> Number of retired breweries: <%= @retired_breweries.count %> </p>

<table class="table table-hover">
  <thead>
  <tr>
    <th>Name</th>
    <th>Year</th>
  </tr>
  </thead>

  <tbody>
  <% @retired_breweries.each do |brewery| %>
      <tr>
        <td><%= link_to brewery.name, brewery %></td>
        <td><%= brewery.year %></td>
        <td></td>
      </tr>
  <% end %>
  </tbody>
</table>

<br>

<%= link_to 'New Brewery', new_brewery_path, class:"btn btn-primary" %>
```

Ratkaisu on toimiva, mutta siinä on parillakin tapaa parantamisen varaa. Parannellaan ensin kontrolleria.

## Scopet

Kontrolleri siis haluaa listan sekä aktiivisista että jo lopettaneista panimoista. Kontrolleri myös kertoo kuinka nuo listat haetaan tietokannasta.

Voisimme tehdä kontrollerista siistimmän, jos luokka <code>Brewery</code> tarjoaisi mukavamman rajapinnan panimoiden listan hakuun. ActiveRecord tarjoaa tähän mukavan ratkaisun, scopet, ks. http://guides.rubyonrails.org/active_record_querying.html#scopes

Määritellään nyt panimoille kaksi scopea, aktiiviset ja eläköityneet:

```ruby
class Brewery < ApplicationRecord
  has_many :beers, dependent: :destroy
  has_many :ratings, through: :beers

  validates :name, presence: true
  validates :year, numericality: { only_integer: true,
                                   greater_than: 1039,
                                   less_than_or_equal_to: ->(_) { Time.now.year } }

  scope :active, -> { where active: true }
  scope :retired, -> { where active: [nil,false] }

  include RatingAverage
end

```

Scope määrittelee luokalle metodin, joka palauttaa kaikki scopen määrittelevän kyselyn palauttamat oliot.

Nyt <code>Brewery</code>-luokalta saadaan pyydettyä kaikkien panimoiden lisäksi mukavan rajapinnan avulla aktiiviset ja lopettaneet panimot:

    Brewery.all      # kaikki panimot
    Brewery.active   # aktiiviset
    Brewery.retired  # lopettaneet

Kontrollerista tulee nyt elegantti:

```ruby
def index
  @active_breweries = Brewery.active
  @retired_breweries = Brewery.retired
end
```

Ratkaisu on luettavuuden lisäksi parempi myös olioiden vastuujaon kannalta. Ei ole järkevää laittaa kontrollerin vastuulle sen kertomista _miten_ aktiiviset ja eläköityneet panimot tulee hakea kannasta, sen sijaan tämä on hyvin luontevaa antaa modelin vastuulle, sillä modelin rooli on nimenomaan toimia abstraktiokerroksena muun sovelluksen ja tietokannan välillä.

Kannattaa huomioida, että ActiveRecord mahdollistaa operaatioiden ketjuttamisen. Voitaisiin kirjoittaa esim:

```ruby
  Brewery.where(active: true).where("year>2000")
```

ja tuloksena olisi SQL-kysely

```sql
SELECT "breweries".* FROM "breweries" WHERE "breweries"."active" = ? AND (year>2000)
```

ActiveRecord osaa siis optimoida ketjutetut metodikutsut yhdeksi SQL-operaatioksi. Myös scope toimii osana ketjutusta, eli vuoden 2000 jälkeen perustetut, edelleen aktiiviset panimot saataisiin selville myös seuraavalla 'onelinerilla':

```ruby
Brewery.active.where("year>2000")
```

## Partiaalit

Siistitään seuraavaksi panimolistan näyttötemplatea. Templatessa on nyt käytännössä sama taulukko kopioituna kahteen kertaan peräkkäin. Eristämme taulukon omaksi __partiaaliksi__, eli näyttötemplateen upotettavaksi tarkoitetuksi näyttötemplaten palaksi, ks. http://guides.rubyonrails.org/layouts_and_rendering.html#using-partials.

Annetaan partialille nimi views/breweries/_list.html.erb (Huom: partialien nimet ovat aina alaviiva-alkuisia!). Sisältö on seuraava:

```erb
<table class="table table-hover">
  <thead>
  <tr>
    <th>Name</th>
    <th>Year</th>
  </tr>
  </thead>

  <tbody>
    <% breweries.each do |brewery| %>
      <tr>
        <td><%= link_to brewery.name, brewery %></td>
        <td><%= brewery.year %></td>
        <td></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

Partiaali viittaa nyt taulukkoon sijoitettavien panimoiden listaan nimellä <code>breweries</code>.

Kaikki panimot renderöivä template ainoastaan *renderöi partiaalin* ja lähettää sille renderöitävän panimolistan parametriksi:

```erb
<h1>Breweries</h1>

<h2>Active</h2>

<p> Number of active breweries: <%= @active_breweries.count %> </p>

<%= render 'list', breweries: @active_breweries %>

<h2>Retired</h2>

<p> Number of retired breweries: <%= @retired_breweries.count %> </p>

<%= render 'list', breweries: @retired_breweries %>

<br>

<%= link_to 'New Brewery', new_brewery_path, class:"btn btn-primary" %>
```

Panimoiden sivun template on nyt lähes silmiä hivelevä!

> ## Tehtävä 6-7 (kahden tehtävän arvoinen)
>
> Ratings-sivumme on tällä hetkellä hieman tylsä. Muuta sivua siten, että sillä näytetään reittausten sijaan:
>* kolme reittausten keskiarvon perusteella parasta olutta ja panimoa
>* viisi viimeksi tehtyä reittausta
>
> **Vihjeitä:**
>
> Jos panimolla/oluella ei ole yhtään ratingia, tuottaa ratingien keskiarvon laskeminen eli metodin <code>average_rating</code> kutsuminen todennäköisesti virheen (järjestäessäsi panimoita reittauksen perusteella). Korjaa metodi siten, että se osaa laskea reittausten keskiarvon myös reittaamattomille oluille/panimoille.
>
> Tee luokalle <code>Rating</code> scope <code>:recent</code>, joka palauttaa viisi viimeisintä reittausta. Scopen vaatimaan tietokantakyselyyn löydät apuja linkistä http://guides.rubyonrails.org/active_record_querying.html, ks. order ja limit. Kokeile ensin kyselyn tekoa konsolista!
>
> Parhaiden oluet ja panimot sekä innokkaimmat reittaajat kertovien scopejen teko ei onnistu yhtä helposti, sillä scopen palauttamat oliot pitäisi selvittää tietokantatasolla eli tarvittaisiin monimutkaista SQL:ää.
>
> Scopejen sijaan voit tehdä luokille <code>Brewery</code>, <code>Beer</code> ja <code>User</code> luokkametodit (eli Javan terminologiassa staattiset metodit), joiden avulla kontrolleri saa haluamansa panimot, oluet ja käyttäjät. Esim. panimolla metodi olisi suunilleen seuraavanlainen:
>
>```ruby
>class Brewery
>  # ...
>
>  def self.top(n)
>    sorted_by_rating_in_desc_order = Brewery.all.sort_by{ |b| -(b.average_rating || 0) }
>    # palauta listalta parhaat n kappaletta
>    # miten? ks. http://www.ruby-doc.org/core-2.5.1/Array.html
>  end
>end
>```
>
> Metodia käytetään nyt kontrollerista seuraavasti:
>
>```ruby
>  @top_breweries = Brewery.top 3
>```
>
> Huom: oluiden, tyylien  ja panimoiden <code>top</code>-metodit ovat oikeastaan copypastea ja moduuleja käyttämällä olisi mahdollista saada koodin määrittely siirrettyä yhteen paikkaan. Kun olet tehnyt viikon kaikki tehtävät voit yrittää siistiä koodisi!
>
> __Älä copypastaa näyttöjen koodia vaan käytä tarvittaessa partiaaleja.__

> ## Tehtävä 8
>
> Lisää reittausten sivulle myös parhaat kolme oluttyyliä ja kolme eniten reittauksia tehnyttä käyttäjää.

Reittausten sivu voi näyttää tehtävävien jälkeen esim. seuraavalta:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w6-4.png)

Sivun muotoiluun voi olla apua seuraavasta: https://getbootstrap.com/docs/4.0/layout/grid/#nesting

## Näyttöjen koodin siistiminen helpereillä

Viikolla 3 lisäsimme luokkaan <code>ApplicationController</code> metodin <code>current_user</code> jonka määrittelimme myös ns. helper-metodiksi

```ruby
class ApplicationController < ActionController::Base
  # ...
  helper_method :current_user

 end
```

näin sekä kontrollerit että näkymät voivat tarvittaessa käyttää metodia kirjaantuneena olevan käyttäjän identiteetin tarkastamiseen. Koska metodi on määritelty luokkaan <code>ApplicationController</code> on se automaattisesti kaikkien kontrollerien käytössä. Helper-metodiksi määrittely tuo metodin myös näkymien käyttöön.

Sovelluksissa on usein tarve kirjoittaa apumetodeja (eli Railsin terminologian mukaan helper-metodeja) pelkästään näyttötemplateja varten. Tällöin niitä ei kannata sijoittaa <code>ApplicationController</code>-luokkaan vaan hakemiston _app/helpers/_ alla oleviin moduuleihin. Jos apumetodia on tarkoitus käyttää useammasta näytöstä, on oikea sijoituspaikka <code>application_helper</code>, jos taas apumetodit ovat tarpeen ainoastaan yhden kontrollerin alaisuudessa olevilla sivuilla, kannattaa ne määritellä ko. kontrolleria vastaavaan helper-moduliin.

Huomaamme, että näyttöjemme koodissa on joitain toistuvia osia. Esim. oluen, tyylin ja panimon show.html.erb-templateissa on kaikissa hyvin samantapainen koodi, jolla sivulle luodaan tarvittaessa linkit editointia ja poistamista varten:

```erb
<% if current_user %>
  <%= link_to 'Edit', edit_brewery_path(@brewery), class:"btn btn-primary"  %>
  <%= link_to 'Destroy', @brewery, method: :delete, data: { confirm: 'Are you sure?' }, class:"btn btn-danger"  %>
<% end %>
```

Eriytetään nämä omaksi helperiksi, moduliin application_helper.rb

```ruby
module ApplicationHelper
  def edit_and_destroy_buttons(item)
    unless current_user.nil?
      edit = link_to('Edit', url_for([:edit, item]), class: "btn btn-primary")
      del = link_to('Destroy', item, method: :delete,
                                     data: { confirm: 'Are you sure?' },
                                     class: "btn btn-danger")
      raw("#{edit} #{del}")
    end
  end
end
```

Metodi muodostaa link_to:n avulla kaksi HTML-linkkielementtiä ja palauttaa molemmat "raakana" (ks. http://apidock.com/rails/ActionView/Helpers/RawOutputHelper/raw), eli käytännössä HTML-koodina, joka voidaan upottaa sivulle.

Painikkeet lisätään esim. oluttyylin sivulle seuraavasti:

```erb
<h2>
  <%= @style.name %>
</h2>

<quote>
  <%= @style.description %>
</quote>

...

<%= edit_and_destroy_buttons(@style) %>
```

Näytön muodostava template siistiytyykin huomattavasti.

Painikkeet muodostava koodi olisi pystytty myös eristämään omaan partialiin, ja onkin hiukan makuasia kumpi on tässä tilanteessa parempi ratkaisu, helper-metodi vai partiali.

> ## Tehtävä 9
>
> Usealla sovelluksen sivulla näytetään reittausten keskiarvoja. Keskiarvot ovat Decimal-tyyppiä, joten ne tulostuvat välillä hieman liiankin monen desimaalin tarkkuudella. Määrittele reittausten keskiarvon renderöintiä varten apumetodi <code>round(number)</code>, joka tulostaa aina parametrinsa __yhden__ desimaalin tarkkuudella, ja ota apumetodi käyttöön (ainakin joissakin) näyttötemplateissa.
>
> Voit käyttää helpperissäsi esim. Railsista löytyvää <code>number_with_precision</code>-metodia, ks. http://api.rubyonrails.org/classes/ActionView/Helpers/NumberHelper.html#method-i-number_with_precision

## Reitti panimon statuksen muuttamiselle

Lisäsimme hetki sitten panimoille tiedon niiden aktiivisuudesta ja mahdollisuuden muuttaa panimon aktiivisuusstatusta panimon tietojen editointilomakkeesta. Kuvitellaan hieman epärealistisesti, että panimot voisivat vähän väliä lopettaa ja aloittaa jälleen toimintansa. Tällöin aktiivisuusstatuksen muuttaminen panimon tietojen editointilomakkeelta olisi hieman vaivalloista. Tälläisessä tilanteessa olisikin kätevämpää, jos esim. kaikkien panimoiden listalla olisi panimon vieressä nappi, jota painamalla panimon aktiivisuusstatuksen muuttaminen onnistuisi. Voisimme toteuttaa tälläisen napin upottamalla panimoiden listalle jokaisen panimon kohdalle sopivan lomakkeen. Teemme kuitenkin nyt toisenlaisen ratkaisun. Lisäämme panimoille Railsin oletusarvoisen kuuden reitin lisäksi uuden reitin <code>toggle_activity</code>, johon tehdyn HTTP POST -kutsun avulla panimon aktiivisuusstatusta voi muuttaa.

Tehdään tiedostoon routes.rb seuraava muutos panimon osalta:

```ruby
resources :breweries do
  post 'toggle_activity', on: :member
end
```

Kun nyt teemme komennon <code>rails routes</code> huomaamme panimolle ilmestyneen uuden reitin:

```ruby
  toggle_activity_brewery POST   /breweries/:id/toggle_activity(.:format)                                                 breweries#toggle_activity
                breweries GET    /breweries(.:format)                                                                     breweries#index
                          POST   /breweries(.:format)                                                                     breweries#create
              new_brewery GET    /breweries/new(.:format)                                                                 breweries#new
             edit_brewery GET    /breweries/:id/edit(.:format)                                                            breweries#edit
                  brewery GET    /breweries/:id(.:format)                                                                 breweries#show
                          PATCH  /breweries/:id(.:format)                                                                 breweries#update
                          PUT    /breweries/:id(.:format)                                                                 breweries#update
                          DELETE /breweries/:id(.:format)                                                                 breweries#destroy
```

Päätämme lisätä aktiivisuusstatuksen muutostoiminnon yksittäisen panimon sivulle. Eli lisätään panimon sivulle app/views/breweries/show.html.erb seuraava:

```erb
<%= link_to "change activity", toggle_activity_brewery_path(@brewery.id), method: :post, class: "btn btn-primary" %>
```

Kun nyt klikkaamme painiketta, tekee selain HTTP POST -pyynnön osoitteeseen /breweries/:id/toggle_activity, missä :id on sen panimon id, jolla linkkiä klikattiin. Railsin reititysmekanismi yrittää kutsua breweries-kontrollerin metodia <code>toggle_activity</code> jota ei ole, joten seurauksena on virheilmoitus. Metodi voidaan toteuttaa esim. seuraavasti:

```ruby
def toggle_activity
  brewery = Brewery.find(params[:id])
  brewery.update_attribute :active, (not brewery.active)

  new_status = brewery.active? ? "active" : "retired"

  redirect_to brewery, notice:"brewery activity status changed to #{new_status}"
end
```

Tominnallisuuden toteuttaminen oli varsin helppoa, mutta onko reitin <code>toggle_activity</code> lisääminen järkevää? RESTful-ideologian mukaan puhdasoppisempaa olisi ollut hoitaa asia lomakkeen avulla, eli polkuun breweries/:id kohdistuneella PUT-pyynnöllä. Jokatapauksessa tulee välttää tilanteita, joissa resurssin tilaa muutettaisiin GET-pyynnöllä, ja tästä syystä määrittelimmekin polun toggle_activity ainoastaan POST-pyynnöille.

Lisää custom routeista sivulla
http://guides.rubyonrails.org/routing.html#adding-more-restful-actions

## Admin-käyttäjä ja pääsynhallintaa

> ## Tehtävä 10
>
> Tällä hetkellä kuka tahansa kirjautunut käyttäjä voi poistaa panimoja, oluita ja olutseuroja. Laajennetaan järjestelmää siten, että osa käyttäjistä on administraattoreja, ja poisto-operaatiot ovat vain sallittuja vain heille
>
> * luo User-modelille uusi boolean-muotoinen kenttä <code>admin</code>, jonka avulla merkataan ne käyttäjät joilla on ylläpitäjän oikeudet järjestelmään
> * riittää, että käyttäjän voi tehdä ylläpitäjäksi ainoastaan konsolista
> * tee panimoiden, oluiden, olutseurojen ja tyylien poisto-operaatioista ainoastaan ylläpitäjälle mahdollinen toimenpide
>
> **Huom:** salasanan validoinnin takia käyttäjän tekeminen adminiksi konsolista ei onnistu, jos salasanakenttiin ei ole asetettu arvoja:
>
>```ruby
>> u = User.first
>> u.admin = true
>> u.save
>   (0.1ms)  rollback transaction
> => false
>```
>
> Yksittäisten attribuuttien arvon muuttaminen on kuitenkin mahdollista validaation kiertävällä metodilla <code>update_attr</code>:
>
> ```ruby
>> u.update_attribute(:admin, true)
> ```
>
> Validointien suorittamisen voi ohittaa myös tallentamalla olion komennolla <code>u.save(validate: false)</code>
>
> **HUOM:** toteutuksessa kannattanee hyödyntää [esifiltteriä](https://github.com/mluukkai/WebPalvelinohjelmointi2018/blob/master/web/viikko4.md#kirjautuneiden-toiminnot)

> ## Tehtävät 11-12 (kahden tehtävän arvoinen)
>
> Toteuta toiminnallisuus, jonka avulla administraattorit voivat sulkea jonkin käyttäjätunnuksen. Sulkeminen voi tapahtua esim. napilla, jonka vain administraattorit näkevät käyttäjän sivulla. Suljetun tunnuksen omaava käyttäjä ei saa päästä kirjautumaan järjestelmään. Yrittäessään kirjautumista, sovellus huomauttaa käyttäjälle että hänen tunnus on suljettu ja kehoittaa ottamaan yhteyttä ylläpitäjiin. Administraattorien tulee pystyä palauttamaan suljettu käyttäjätunnus ennalleen.
>
> Voit toiteuttaa toiminnallisuuden esim. allaolevien vihjaamaan kuvien tapaan

Administraattori voi sulkea käyttäjätunnuksen käyttäjän sivulta

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w6-1c.png)

Administraattori näkee käyttäjien näkymästä suljetut käyttäjätunnukset

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w6-1b.png)

Jos käyttjätunnus on suljettu, kirjautuminen ei onnistu

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w6-1x.png)

Administraattori voi uudelleenaktivoida suljetun käyttäjätunnuksen käyttäjän sivulta

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w6-1d.png)

## Monimutkaisempi pääsynhallinta

Jos sovelluksessa on tarvetta monipuolisempaan pääsynhallintaan (engl. authorization), kannattanee asia hoitaa esim. _cancan_-gemin avulla ks. https://github.com/CanCanCommunity/cancancan ja
http://railscasts.com/episodes/192-authorization-with-cancan

Aihetta esittelevä Rails cast on jo aika ikääntynyt, eli tarkemmat ohjeet kannattaa katsoa projektin Github-sivulta. Rails castit tarjoavat todella hyviä esittelyjä monista aihepiireistä, eli vaikka castit eivät enää olisi täysin ajantasalla kaikkien detaljien suhteen, kannattaa ne usein silti katsoa läpi.

## Rails-sovellusten tietoturvasta

Emme ole vielä toistaiseksi puhuneet mitään Rails-sovellusten tietoturvasta. Nyt on aika puuttua asiaan. Rails-guideissa on tarjolla erinomainen katsaus tyypillisimmistä web-sovellusten tietoturvauhista ja siitä miten Rails-sovelluksissa voi uhkiin varautua.

> ## Tehtävät 13-15 (kolmen tehtävän arvoinen)
>
> Lue https://guides.rubyonrails.org/security.html
>
> Teksti on pitkä mutta asia on tärkeä. Jos haluat optimoida ajankäyttöä, jätä luvut 4, 5, 7.4-7.8, 8, 9 ja 10 lukematta.
>
> Voit merkata tehtävät tehdyksi kun seuraavat asiat selvillä
> * SQL-injektio
> * CSRF
> * XSS
> * järkevä sessioiden käyttö
>
> Tietoturvaan liittyen kannattaa katsoa myös seuraavat
> * https://guides.rubyonrails.org/action_controller_overview.html#force-https-protocol
> * https://guides.rubyonrails.org/action_controller_overview.html#log-filtering

Ylläolevasta dokumentista ei käy täysin selväksi se, että Rails _sanitoi_ (eli escapettaa kaikki script- ja html-tagit yms) oletusarvoisesti sivuilla renderöitävän syötteen, eli esim. jos yrittäisimme syöttää javascript-pätkän <code> &lt;script&gt;alert(&#39;Evil XSS attack&#39;);&lt;/script&gt;</code> oluttyylin kuvaukseen, koodia ei suoriteta, vaan koodi renderöityy sivulle 'tekstinä':

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w5-7.png)

Jos katsomme sivun lähdekoodia, huomaamme, että Rails on korvannut HTML-tägit aloittavat ja sulkevat < -ja > -merkit niitä vastaavilla tulostuvilla merkeillä, jolloin syöte muuttuu selaimen kannalta normaaliksi tekstiksi:

```ruby
 &lt;script&gt;alert(&#39;Evil XSS attack&#39;);&lt;/script&gt;
```

Oletusarvoisen sanitoinnin saa 'kytkettyä pois' pyytämällä eksplisiittisesti metodin <code>raw</code> avulla, että renderöitävä sisältö sijoitetaan sivulle sellaisenaan. Jos muuttaisimme tyylin kuvauksen renderöintiä seuraavasti

```ruby
<p>
  <%= raw(@style.description) %>
</p>
```

suoritetaan javascript-koodi sivun renderöinnion yhteydessä:

![kuva](https://github.com/mluukkai/WebPalvelinohjelmointi2018/raw/master/images/ratebeer-w5-8.png)

Lisätietoa http://www.railsdispatch.com/posts/security ja http://railscasts.com/episodes/204-xss-protection-in-rails-3

## Mielipanimoiden ja tyylin refaktorointi

Tällä viikolla ei ole enää enempää tehtäviä. Riittää että luet tästä eteenpäin olevan materiaalin. Seuraavan viikon materiaali ei riipu millään tavalla tämän viikon päättävästä refaktoroinnista.

Viikon 4 [tehtävissä 3 ja 4](https://github.com/mluukkai/WebPalvelinohjelmointi2018/blob/master/web/viikko4.md#teht%C3%A4v%C3%A4-3)  toteutettiin metodit henkilön suosikkipanimon ja oluttyylin selvittämiseen. Seuraavassa on eräs melko suoraviivainen ratkaisu metodien <code>favorite_style</code> ja <code>favorite_brewery</code> toteuttamiseen:

```ruby
class User
  # ...
  def favorite_style
    return nil if ratings.empty?

    style_ratings = ratings.group_by{ |r| r.beer.style }
    averages = style_ratings.map do |style, ratings|
      { style: style, score: average_of(ratings) }
    end

    averages.max_by{ |r| r[:score] }[:style]
  end

  def favorite_brewery
    return nil if ratings.empty?

    brewery_ratings = ratings.group_by{ |r| r.beer.brewery }
    averages = brewery_ratings.map do |brewery, ratings|
      { brewery: brewery, score: average_of(ratings) }
    end

    averages.max_by{ |r| r[:score] }[:brewery]
  end

  def average_of(ratings)
    ratings.sum(&:score).to_f / ratings.count
  end  
end  
```

Tutkitaan mielipanimon selvittävää metodia:

```ruby
def favorite_brewery
  return nil if ratings.empty?

  brewery_ratings = ratings.group_by{ |r| r.beer.brewery }
  averages = brewery_ratings.map do |brewery, ratings|
    { brewery: brewery, score: average_of(ratings) }
  end

  averages.max_by{ |r| r[:score] }[:brewery]
end
```

Erikoistapauksen (ei yhtään reittausta) tarkastamisen jälkeen metodi ryhmittelee [group_by](https://ruby-doc.org/core-2.5.1/Enumerable.html#method-i-group_by)-metodin avulla ratingit niihin liittyvän panimo mukaan:

```ruby
brewery_ratings = ratings.group_by{ |r| r.beer.brewery }
```

Operaatio tuottaa _hashin_ jonka avaimina ovat panimot, joita käyttäjä on reitannut ja arvona avainta vastaavan panimon saamat käyttäjän tekemät reittaukset.

Hash näyttää suunilleen seuraavalta

```
{
  { name: "koff" }: [ 
    { score: 10, beer_id: 3, user_id: 1 }, { score: 17, beer_id: 4, user_id: 1 }
  ],
  { name: "karjala" }: [ 
    { score: 20, beer_id: 7, user_id: 1 }, { score: 40, beer_id: 11, user_id: 1 }, { score: 9, beer_id: 8, user_id: 1 }
  ],  
  { name: "weihenstephan" }: [ 
    { score: 44, beer_id: 12, user_id: 1 }
  ],   
}
```

Seuraava komento

```ruby
averages = brewery_ratings.map do |brewery, ratings|
  { brewery: brewery, score: average_of(ratings) }
end
```

muodostaa taulukon, minkä alkioina on hashejä, jotka sisältävät panimon ja siihen liittyvien reittausten keskiarvon.

```
[
  { 
    brewery: { name: "koff" },
    score: 17
  },
  { 
    brewery: { name: "karjala" },
    score: 21
  },
  { 
    brewery: { name: "weihenstephan" },
    score: 40
  }    
]
```

Metodin viimeinen rivi valitsee taulukon alkioista sen, jonka _score_ on suurin ja palauttaa alkioon liittyvän panimon:

```ruby
averages.max_by{ |r| r[:score] }[:brewery]
```

Lempityylin selvittävä metodi on rakenteellisesti täysin samanlainen ja saamme pienellä refaktoroinnilla rakennettua yleistetyn koodin, jonka avulla pystymme selvittämään sekä lempipanimon että tyylin.

Muutetaan ensin metodien käyttäjien apumuuttujien ja hash-avainten arvo vastaamaan toisiaan

```ruby
def favorite_style
  return nil if ratings.empty?

  grouped_ratings = ratings.group_by{ |r| r.beer.style }
  averages = grouped_ratings.map do |group, ratings|
    { group: group, score: average_of(ratings) }
  end

  averages.max_by{ |r| r[:score] }[:group]
end

def favorite_brewery
  return nil if ratings.empty?

  grouped_ratings = ratings.group_by{ |r| r.beer.brewery }
  averages = grouped_ratings.map do |group, ratings|
    { group: group, score: average_of(ratings) }
  end

  averages.max_by{ |r| r[:score] }[:group]
end
```

Testit menevät edelleen läpi, eli toiminnallisuus ei ole muuttunut. Molemmat metodit ovat nyt lähes samanlaiset, ainoa ero on _group_by_-metodiin liittyvässä koodilohkossa

```ruby
grouped_ratings = ratings.group_by{ |r| r.beer.style }
grouped_ratings = ratings.group_by{ |r| r.beer.brewery }
```

Saamme nämäkin rivin täsmälleen samanlaisiksi kutsumalla metodia epäsuoraan viime viikolta tuttua [send](https://github.com/mluukkai/WebPalvelinohjelmointi2018/blob/master/web/viikko5.md#olion-metodien-kutsuminen-send-metodin-avulla)-metodia hyödyntäen:

```ruby
def favorite_style
  groupped_by = :style
  return nil if ratings.empty?
  
  grouped_ratings = ratings.group_by{ |r| r.beer.send(groupped_by) }
  averages = grouped_ratings.map do |group, ratings|
    { group: group, score: average_of(ratings) }
  end

  averages.max_by{ |r| r[:score] }[:group]
end

def favorite_brewery
  groupped_by = :brewery
  return nil if ratings.empty?

  grouped_ratings = ratings.group_by{ |r| r.beer.send(groupped_by) }
  averages = grouped_ratings.map do |group, ratings|
    { group: group, score: average_of(ratings) }
  end

  averages.max_by{ |r| r[:score] }[:group]
end
```

Testien suoritus antaa jälleen varmuuden siitä että toiminnallisuus pysyy muuttumattoman.

Metodit ovat nyt täysin samat lukuunottamatta muutujan _groupped_by_ arvoa. Siirretään yhteinen logikka omaan metodiin

```ruby
def favorite_style
  favorite(:style)
end

def favorite_brewery
  favorite(:brewery)
end

def favorite(groupped_by)
  return nil if ratings.empty?

  grouped_ratings = ratings.group_by{ |r| r.beer.send(groupped_by) }
  averages = grouped_ratings.map do |group, ratings|
    { group: group, score: average_of(ratings) }
  end

  averages.max_by{ |r| r[:score] }[:group]
end
```

Testit menevät edelleen läpi ja copypaste on poissa!

Uuden ratkaisumme etu on copypasten poiston lisäksi se, että jos oluelle määritellään jokun uusi "attribuutti", esim. väri, saamme samalla hinnalla mielivärin selvittävän metodin:

```ruby
  def favorite_color
    favorite :color
  end
```

## method_missing

Metodit <code>favorite_style</code> ja <code>favorite_brewery</code> olisi oikeastaan mahdollista saada toimimaan ilman niiden eksplisiittistä määrittelemistä.

Kommentoidaan metodit hetkeksi pois koodistamme.

Jos oliolle kutsutaan metodia, jota ei ole olemassa (määriteltynä luokassa itsessään, sen yliluokissa eikä missään luokan tai yliluokkien sisällyttämässä moduulissa), esim.

```ruby
> u = User.first
> u.paras_bisse
NoMethodError: undefined method `paras_bisse' for #<User:0x00007f9b415269d8>
from /Users/mluukkai/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/activemodel-5.2.1/lib/active_model/attribute_methods.rb:430:in `method_missing'b:69:in `console'
>
```

on tästä seurauksena se, että Ruby-tulkki kutsuu olion <code>method_missing</code>-metodia parametrinaan tuntemattoman metodin nimi. Rubyssä kaikki luokat perivät <code>Object</code>-luokan, joka määrittelee <code>method_missing</code>-metodin. 

Luokkien on sitten tarvittaessa mahdollista ylikirjoittaa tämä metodi ja saada näinollen aikaan "metodeja" joita ei ole olemassa, mutta jotka kutsujan kannalta toimivat aivan kuten normaalit metodit.

Rails käyttää sisäisesti metodia <code>method_missing</code> moniin tarkoituksiin. Emme voikaan suoraviivaisesti ylikirjoittaa sitä, meidän on muistettava delegoida <code>method_missing</code>-kutsut yliluokalle jollemme halua käsitellä niitä itse.

Määritellään luokalle <code>User</code> kokeeksi seuraavanlainen <code>method_missing</code>:

```ruby
def method_missing(method_name, *args, &block)
  puts "nonexisting method #{method_name} was called with parameters: #{args}"
  return super
end
```

kokeillaan:

```ruby
> u.paras_bisse
nonexisting method paras_bisse was called with parameters: []
NoMethodError: undefined method `paras_bisse' for #<User:0x00007f9b41c02ef0>
from /Users/mluukkai/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/activemodel-5.2.1/lib/active_model/attribute_methods.rb:430:in `method_missing'
```

Eli kuten ylimmältä riviltä huomataan, suoritettiin määrittelemämme <code>method_missing</code>-metodi. Voimmekin ylikirjoittaa method_missingin seuraavasti:

```ruby
def method_missing(method_name, *args, &block)
  if method_name =~ /^favorite_/
    category = method_name[9..-1].to_sym
    self.favorite category
  else
    return super
  end
end
```

Nyt kaikki <code>favorite_</code>-alkuiset metodikutsut joita ei tunneta tulkitaan siten, että alaviivan jälkeinen osa eristetään ja kutsutaan oliolle metodia <code>favorite</code>, siten että alaviivan jälkiosa on kategorian määrittelevänä parametrina.

Nyt metodit <code>favorite_brewery</code> ja <code>favorite_style</code> "ovat olemassa" ja toimivat:

```ruby
> u = User.first
> u.favorite_brewery.name
 => "Malmgard"
> u.favorite_style.name
  => "Baltic porter"
```

Ikävänä sivuvaikutuksena metodien määrittelystä method_missing:in avulla  on se, että mikä tahansa favorite_-alkuinen metodi "toimisi", mutta aiheuttaisi kenties epäoptimaalisen virheen.

```ruby
> u.favorite_movie
NoMethodError: undefined method `movie' for #<Beer:0x00007f9b408599f8>
from /Users/mluukkai/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/activemodel-5.2.1/lib/active_model/attribute_methods.rb:430:in `method_missing'
```

Ruby tarjoaa erilaisia mahdollisuuksia mm. sen määrittelemiseen, mitkä <code>favorite_</code>-alkuiset metodit hyväksyttäisiin. Voisimme esim. toteuttaa seuraavan rubymäisen tavan asian määrittelemiselle:

```ruby
class User < ApplicationRecord
  include RatingAverage

  favorite_available_by :style, :brewery

  # ...
end
```

Emme kuitenkaan lähde nyt tälle tielle. Hyöty tulisi näkyviin vasta jos favorite_-alkuisia metodeja voitaisiin hyödyntää muissakin luokissa.

Poistetaan kuitenkin nyt tässä tekemämme method_missing:iin perustuva toteutus ja palautetaan luvun alussa poiskommentoidut versiot.

Jos tässä luvussa esitellyn tyyliset temput kiinnostavat, voit jatkaa esim. seuraavista:

* http://ruby-metaprogramming.rubylearning.com/
* https://github.com/sathish316/metaprogramming_koans
* myös kirja [Eloquent Ruby](http://www.amazon.com/Eloquent-Ruby-Addison-Wesley-Professional-Series/dp/0321584104) käsittelee aihepiiriä varsin hyvin

## Tehtävien palautus

Commitoi kaikki tekemäsi muutokset ja pushaa koodi Githubiin. Deployaa myös uusin versio Herokuun. Muista myös testata rubocopilla, että koodisi noudattaa edelleen määriteltyjä tyylisääntöjä. Tämän viikon esimerkkeihin on tarkoituksella jätetty muutama rubycopin sääntöjä rikkova kohta.

Tehtävät kirjataan palautetuksi osoitteeseen https://studies.cs.helsinki.fi/courses/#/rails2018
