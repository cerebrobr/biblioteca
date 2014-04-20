##Wordpress WP_Query
---

A classe `WP_Query` localizada em ``wp-includes/query.php`` retornas posts com base no que foi setado na sua configuração, também de acordo com a url que esteja sendo visitada, ou seja, a cada requisição o wordpress instancia um novo objeto `WP_Query` de acordo com a pagina que esteja sendo visualisada seja ela categoria, post, página, busca etc. 

Você pode usar funções como `query_posts` ou `get_posts` porém essas mesmas funções iram instanciar a classe WP_Query passando os argumentos setados pelo usuário.

Para visualizar defina esta variável como global, assim poderá depurar seu conteúdo
```php
// note apenas como global você poderá acessar o conteúdo da variável.
global $wp_query; exit(var_dump($wp_query));
```
---
Dependendo da página que você estiver visitando, será possível os query_vars referentes ao conteúdo.

Existem formas de alterar estes `` query_vars``, até mesmo antes o que o wordpress seja carregado por completo.

Caso você esteja dentro do loop e queira usar o objeto ``WP_Query`` ou informações que vieram com o carregamento da página (usado geralmente para artigos relacionados que você precisará do is do post "parent" uma forma de obte-lo é usando ``$wp_query->get_queried_object()``, entre outros), você terá de salvar o estado atual da query, reseta-la logo após re-definir o estado original setado pela página a ser visualisada uma maneira de faze-lo é:

```php
global $wp_query;
// assumindo que você esteja visualizando uma categoria

// crie uma variável temporária para guardar o estado atual da query
$temp = $wp_query;

// logo após resete a variável global $wp_query
$wp_query = null;

// em seguida já com o objeto "limpo" você pode criar sua nova query
// (array) $args array contendo query vars
$wp_query = new WP_Query( $args );

// intere com seu loop personalizado
// o retorno será referente aos query_vars passados anteriormente com $args
if( have_posts() ) :
    while( have_posts() ) :
        the_post();
    endwhile;
endif;

// uma vez que você tenha recuperado os posts você precisa voltar a query para seu estado original, usando a variável $temp que contém nossa query original, referente a página a ser visualisada.

$wp_query = $temp;

```
---
Você também pode manipular a query antes mesmo que os posts sejam retornados pela query, é muito usado em casos em que você deve listar mais de 1 tipo de post, ou posts de 2 categorias diferentes.
Existe uma ação(hook) muito legal, se chama ``pre_get_posts`` com ele você altera a query antes que o wordpress defina o que deve ser exibido, como ele faz isso de acordo coma url. Esta ação nos dá flexibilidade de modo que podemos fazer coisas muito legais com WP.
Ele passa **$query** como parâmetro, o que no nosso caso é o que "virá" em `WP_Query`
A maneira de se usar o `` pre_get_posts ``  é a seguinte:

```php
add_action( 'pre_get_posts', function($query){
    // a linha abaixo irá mostrar o mesmo da global $wp_query
    //exit(var_dump($query));
    
    // primeira verificação é se estamos trabalhando em main_query
    // para isso o WP tem um método chamado is_main_query()
    // uma vez que é nela que desejamos trabalhar
     if ( $query->is_main_query() ) :
        // caso o usuário esteja visualisando o archive do custom post type "bikes"
        // será alterado para "cars"
        if (  is_post_type_archive( 'bikes' ) ) :
            // usamos o método set da classe WP_Query 
            // para definir os query vars desejados um a um
            $query->set( 'post_type', 'cars' );
        endif;
    endif;
});
```
