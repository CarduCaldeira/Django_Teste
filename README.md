# Django: Desenvolvimento guiado por testes

Neste markdown irei documentar o curso "Django: Desenvolvimento guiado por testes" da Alura enquanto realizo ele.

O projeto em que serão aplicados os conceitos estão nesta pasta Django_Teste. 

Para não nos perdemos entre as modificações no projeto e esta documentação, todo commmit dado será reportado aqui. A ideia é que tal documento siga o curso de forma linear.

## Preparando o ambiente

Versões utilizadas

- Python: 3.10.4
## Primeiros passos

Instale um ambiente virtual 
```
python3 -m venv ./venv
```
Para ativar 
```
source /home/carlos/Documentos/Django_Teste/venv/bin/activate
```
Para instalar o Django de o comando
```
pip install Django
```
E para iniciar o projeto com nome setup de o comando 
```
django-admin startproject setup .
```
Aqui darei o primeiro commit com nome "primeiros passos Django:Teste".

# Iniciando testes

Apesar de não termos escrito nenhum teste, o Django possui um comando que roda os testes do nosso projeto
```
python manage.py test
```
Sempre que executamos nossos testes com **manage.py test** é criado um banco de dados. Quando nossos testes acabam esse banco de dados é destruído. O Django cria esse banco de dados pra manter a integridade e consistência dos dados de produção.

Vamos iniciar a construção dos nosos testes. Crie um arquivo **tests.py** em setup. Para subir um servidor de teste importe
```
from django.test import LiveServerTestCase
```
Para simular as funcionalidades da nossa aplicação (como por exemplo abrir determinada pagina, clicar em determinado botão etc) vamos instalar o selenium
```
pip install selenium
```
e importe no arquivo de teste
```
from selenium import webdriver
```
Na classe **AnimaisTestCase** vamos implementar os teste para um dos nossos aplicativos.

```
class AnimaisTestCase(LiveServerTestCase):

    def setUp(self):
        self.browser = webdriver.Chrome('/home/carlos/Documentos/Django_Teste/chromedriver')

    def tearDown(self):
        self.browser.quit()
```
Note que em **setUp** estamos iniciando nosso navegador(estamos adotando o Chrome). O arquivo **chromedriver** pode ser obtido em https://selenium-python.readthedocs.io/installation.html#drivers.

A função **tearDown** é responsavel por fechar o navegador. 

Como exemplo poderíamos implementar uma função que abre o a url def https://www.google.com/.
Ela ficaria da seguinte forma
```
def test_para_verificar_se_a_janela_do_browser_esta_ok(self):
        self.browser.get('https://www.google.com/')
```
É importante que comecemos o nome da função com **test_**.

Como queremos testar a nossa aplicação poderiamos implementar uma função que abre a nossa home page 
```
def test_abre_janela_do_chrome(self):
        self.browser.get(self.live_server_url)
```
Para ser mais semantico que é a nossa home page poderia ser 
```
def test_abre_janela_do_chrome(self):
        self.browser.get(self.live_server_url+'/')
```
Como a função **test_abre_janela_do_chrome** foi só um exemplo voce pode deleta-la.

# Teste buscando um novo animal

Já que montamos a estrutura para os nossos testes, vamos implementar o teste para 
a funcionalidade que retorna se uma animal é domestico, venenoso, etc. O usuario seguira o seguinte roteiro
- Vini deseja encontrar um novo animal para adotar.
- Ele encontra o Busca Animal e decide usar o site, porque ele vê no menu do site escrito Busca Animal.
- Ele vê um campo para pesquisar animais pelo nome. 
- Ele pesquisa por Leão e clica no botão pesquisar.
- O site exibe 4 caracteristicas do animal pesquisado.
- Ele desiste de adotar um leão.

Assim  crie um metodo com nome **test_buscando_um_novo_animal** para implemmentar tais passos,
```
    def test_buscando_um_novo_animal(self):
        home_page = self.browser.get(self.live_server_url + '/')
        brand_element = self.browser.find_element_by_css_selector('.navbar')
        self.assertEqual('Busca Animal', brand_element.text)
```
Primeiro abrimos a home page, depois comparamos o conteudo do navbar com 'Busca Animal'. Para usar o comando **find_element(By.CSS_SELECTOR, '.navbar')** é necessario importar
```
from selenium.webdriver.common.by import By
```

O proximo passo é construir o app animais, em que implementaremos o código a ser testado.
Assim, de o comando no terminal
```
python manage.py startapp animais
```
Exclua o arquivo **test.py**. Adicione em **settings.py** em **INSTALLED_APPS**, o app animais. Também crie uma pasta 
tests em animais e adicione dois arquivos .py nela, o **__init__.py** e o **tests_url.py**,
neste ultimo arquivo iremos checar se determinada requisição é atendida por uma view especifica.

Neste arquivo importe 
```
from urllib import request
from django.test import TestCase, RequestFactory
from animais.views import index
```
Utilizaremos o TestCase para realizar os testes, o RequestFactory para identificar qual a url que estamos utilizando e o index que é a view da home page. Crie a classe **AnimaisURLSTestCase** como abaixo
```
class AnimaisURLSTestCase(TestCase):

    def setUp(self):
        self.factory = RequestFactory() 

    def test_rota_url_utiliza_view_index(self):
        """Teste se a home da aplicação utiliza a função index da view"""
        request = self.factory.get('/')
        response = index(request)
        self.assertEqual(response.status_code,200)
```
O nosso test_rota_url_utilizar_view_index testa se a home da aplicação utiliza a função index da view.
Assim, precisamos criar o index em views. 
```
from django.shortcuts import HttpResponse

# Create your views here.

def index(request):
    return HttpResponse()
```
E registra-la em setup.urls 
```
from animais.views import index
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', index)
]
```

# Adicionando o template

Implementamos a view index retornando um HttpResponse, porém queremos implementar um templante **index.html**. Vamos fazer um teste pra isso. Então na class atualize **test_rota_url_utiliza_view_index**
```
def test_rota_url_utiliza_view_index(self):
        """Teste se a home da aplicação utiliza a função index da view"""
        request = self.factory.get('/')
        with self.assertTemplateUsed('index.html'):
            response = index(request)
            self.assertEqual(response.status_code,200)
```
O comando with self.assertTemplateUsed('index.html') ira determinar se estamos usando determinado template.
Assim, no app Animais crie uma pasta templates, e nela crie o arquivo index.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Busca Animal</title>
</head>
<body>
    <a class="navbar"> Busca Animal</a>
</body>
</html>
```
E em views atualize para
```
from django.shortcuts import render

# Create your views here.

def index(request):
    return render(request,'index.html')
```
# Teste para o campo pesquisar

O proximo passo que o usuario realizara é ver o campo pesquisar e nele inserir Leão e clicar no botão pesquisar. Em **setup.tests** insira na classe **test_buscando_um_novo_animal**
```
buscar_animal_input = self.browser.find_element(By.CSS_SELECTOR, 'input#buscar-animal')
self.assertEqual(buscar_animal_input.get_attribute('placeholder'),'Exemplo: leão, urso ...')
```
Assim,  estamos checando se no campo pesquisar aparece escrito como sugestao 'Exemplo: leão, urso ...'(placeholder).

Apos isto, vamos simular o usuario inserindo leao e clicando o botão pesquisar. Entao como ja selecionamos o botão na variavel **buscar_animal_input**, basta inserir os comandos
```
buscar_animal_input.send_keys('leao')
self.browser.find_element(By.CSS_SELECTOR,'form button').click()
```
Apos o usuario clicar nosso programa deve retornar 4 caracteristicas do animal inserido, então insira
```
caracteristicas = self.browser.find_elements(By.CSS_SELECTOR,'.result-description')
self.assertGreater(len(caracteristicas),3)
```
que checara se foi dado mais que tres caracteristicas. Note no codigo anterior que usamos **find_elements** no plural. Assim, no **index.html** insira abaixo do form quatro vezes o codigo
```
<div class="result-description"></div>
```
Se rodarmos os teste note que não teremos erros, porém não inserimos nenhuma caracteristica dentro dos div's.

Então o próximo passo é validar as caracteristicas. Crie um arquivo **test_views.py** em animais.tests
e insira
```
from django.test import TestCase, RequestFactory
from django.db.models.query import QuerySet

class IndexViewTestCase(TestCase):

    def setUp(self):
        self.factory=RequestFactory()

    def test_index_view_retorna_caracteristicas(self):
        """ Teste que verifica se a index retorna as caracteristicas"""

        response = self.client.get('/',{'caracteristicas':'resultado'})
        self.assertIs(type(response.context['caracteristicas']),QuerySet)
```
Na metodo **test_index_view_retorna_caracteristicas** estamos verificando se  o tipo deste nosso resultado é uma QuerySet. Por exemplo se deixarmos a nossa view index da seguinte forma
```
def index(request):
    context = {'caracteristicas':None}
    return render(request,'index.html',context)
```
e rodarmos um teste, teremos um erro informando que o tipo NoneType não é um QuerySet. Porém o que deveriamos colocar no lugar de None? Perceba aqui que poderiamos pegar tais caracteristicas do banco de dados.

## Teste para o Models

Como queremos utilizar o banco de dados vamos modificar o metodo  index da view  e deixa-la da seguinte forma
```
from django.shortcuts import render
from animais.models import Animal

# Create your views here.

def index(request):
    context = {'caracteristicas': Animal.objects.all()}
    return render(request,'index.html',context)

```
Então precisamos criar essa classe Animal em animais.models 
```
from django.db import models

# Create your models here.

class Animal(models.Model):
    nome_animal = models.CharField(max_length=20)
    predador = models.CharField(max_length=5)
    venenoso = models.CharField(max_length=5)
    domestico = models.CharField(max_length=5)

    def __str__(self):
        return self.nome_animal
```
E também iremos criar um arquivo **test_models.py** em animais.tests para testar a classe Animal(teste unitario) em models.
```
from django.test import TestCase, RequestFactory
from animais.models import Animal

class AnimalModelTestCase(TestCase):

    def setUp(self):
        self.animal = Animal.objects.create(
            nome_animal = 'Leão',
            predador = 'Sim',
            venenoso = 'Não',
            domestico = 'Não'
        )
    
    def test_animal_cadastrado_com_caracteristicas(self):
        """Teste que verifica se o animal está cadastrado com suas respectivas características"""
        self.assertEqual(self.animal.nome_animal, 'Leão')
```
E vamos atualizar **test_views.py** para realizar o teste que tinhamos iniciado.
```
from django.test import TestCase, RequestFactory
from django.db.models.query import QuerySet
from animais.models import Animal

class IndexViewTestCase(TestCase):

    def setUp(self):
        self.factory=RequestFactory()
        self.animal = Animal.objects.create(
            nome_animal = 'calopsita',
            predador = 'Não',
            venenoso = 'Não',
            domestico ='Sim'
        )

    def test_index_view_retorna_caracteristicas(self):
        """ Teste que verifica se a index retorna as caracteristicas"""

        response = self.client.get('/', {'buscar':'calopsita'})
        caracteristica_animal_pesquisado = response.context['caracteristicas']
        self.assertIs(type(response.context['caracteristicas']),QuerySet)
        self.assertEqual(caracteristica_animal_pesquisado[0].nome_animal,'calopsita' )
```
Primeiro instanciamos um objeto da classe model com nome calopsita, no metodo **test_index_view_retorna_caracteristicas** mandamos um requisição GET para a nossa home principal,
no comando abaixo caracteristica_animal_pesquisado recebe  Animal.objects.all() e então fazemos a verificação que gostariamos. 

Em "response = self.client.get('/')" poderiamos escrever
```
response = self.client.get('/', {'buscar':'calopsita'})
```
E no input do index.html insirir **name="buscar"**. Assim, estariamos simulando o usuario inserindo calopsita no pesquisar. Por enquanto isso não fara diferença, porem utilizaremos isso nos proximos passos.

## Terminando o teste funcional

Agora iremos terminar o **test.py** em **setup**. Observe que no nosso projeto não estamos mostrando
as caracteristicas do animal digitado, então  devemos integrar com o banco de dados e modificar o index.html e a sua view. Assim no **test.py** em **SetUp** adicione 
```
self.animal = Animal.objects.create(nome_animal = 'leao',
                                    predador = 'Sim',
                                    venenoso = 'Nao',
                                    domestico = 'Nao')
```
Em **views.py** atualize para 
```
from django.shortcuts import render
from animais.models import Animal

# Create your views here.

def index(request):
    context = {'caracteristicas':None}

    if 'buscar' in request.GET:
        animais  = Animal.objects.all()
        animal_pesquisado = request.GET['buscar']
        caracteristicas = animais.filter(nome_animal__icontains = animal_pesquisado)
        context = {'caracteristicas': caracteristicas}
    return render(request,'index.html',context)
```
E em **index.html** atualize as div's para 
```
{% for caracteristica in caracteristicas %}
        <div class="result-description">{{caracteristica.nome_animal}}</div>
        <div class="result-description">{{caracteristica.predador}}</div>
        <div class="result-description">{{caracteristica.venenoso}}</div>
        <div class="result-description">{{caracteristica.domestico}}</div>
{% endfor %}
```
Relembre que para rodar o runserver e nosso projeto não apresentar erros devemos dar os comandos makemigrations e migrate. Agora vamos popular o nosso banco de dados. Com o arquivo **lista_animais.py** rode ele e ele salvara uma lista de animais.

Também vamos estilizar nosso projeto, crie uma pasta static em setup e nele insira **style.css**. Portanto devemos configurar os settings com
```
STATIC_ROOT = os.path.join(BASE_DIR,'static')
STATICFILES_DIRS = [os.path.join(BASE_DIR,'setup/static')]
```
E por ultimo vamos atualizar o **index.html** deixando ele da seguinte forma
```
{% load static%}
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" type="text/css" href="{% static 'style.css' %}" media="screen" />
  <title>Busca animal</title>
</head>
<body>
  

<nav class="navbar">
  <a class="navbar__link" href="#">Busca Animal</a>
</nav>

<section class="search-section">
    <h1 class="title">Pesquise um animal e descubra <span class="title__detail">4 coisas</span> sobre ele</h1>

    <form class="form">
      <input class="form__input" type="text" class="form-control" id="buscar-animal" name="buscar" placeholder="Exemplo: leão, urso..." aria-label="Buscar animal"/>
        
      <button type="submit" class="button">Pesquisar</button>
    </form>
</section>
    {% for caracteristica in caracteristicas%}
<section class="results-section">
  <ul class="results">
    <li class="result">
      <h3 class="result-description">
        Nome do animal
      </h3>
      <h2 class="result-title">
        {{caracteristica.nome_animal}}
      </h2>
    </li>
    <li class="result">
      <h2 class="result-description">
        Predador
      </h2>
      <h3 class="result-description">
        {{caracteristica.predador}}
      </h3>
    </li>
    <li class="result">
      <h2 class="result-description">
        Venenoso
      </h2>
      <h3 class="result-description">
        {{caracteristica.venenoso}}
      </h3>
    </li>
    <li class="result">
      <h2 class="result-description">
        Doméstico
      </h2>
      <h3 class="result-description">
        {{caracteristica.domestico}}
      </h3>
    </li>
  </ul>
</section>
{% endfor %}
        
</body>
</html>
```
E de o comando no terminal
```
python manage.py collectstatic
```
Por fim, podemos rodar python manage.py test e ver que todos os testes o, e tambem rodar o python manage.py runserver.



    


