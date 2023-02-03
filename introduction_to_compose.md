# Uma Introdução ao Compose

O **Compose** é o novo sistema de UI do Android. Constrói e atualiza a tela em ciclos de *recomposição*, onde ele otimamente monta uma árvore de componentes e consegue identificar onde ocorreu uma mudança de estado para recompor somente os elementos afetados pelo estado alterado.

**Composable**: é a forma de declararmos funções que geram UI para o Compose. Essas funções não possuem retorno, mas podem receber parâmetros. Todo composable deve ser anotado com *@Composable*.

Os composables devem ser funções idempotentes, isto é, pra um mesmo conjunto de parâmetros de entrada devem gerar um mesmo composable (ou seja, devemos tratar os parâmetros para gerar UI, e não usar variáveis globais, valores randômicos ou contadores dentro de um composable).

Aqui temos um exemplo de como setar o compose numa activity:

```kotlin
class MainActivity : ComponentActivity() {
     override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
           HelloWorldAppTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colors.surface
                ) {
                    HelloWorldScreen()
                }
            }
        }
    }
}

@Composable
fun HelloWorldScreen() {
    Column {
        Text(“Hello World”)
    }
}
```
<br/>

Pra colocar o compose num Fragment, o processo se dá no *onCreateView*

```kotlin
class MyFragment: Fragment() {
    [...]
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View { 
        return ComposeView(requireContext()).apply {
            setViewCompositionStrategy(ViewCompositionStrategy.DisposeOnViewTreeLifecycleDestroyed)
            setContent {
                HelloWorldAppTheme {
                    HelloScreen()
                }
            }
        }
    }

    [...]
}

@Composable
fun HelloScreen() {
    Column {
        Text("Hello World")
    }
}
```
<br/>

> Obs: é prática comum que o primeiro composable de uma *Activity* ou *Fragment* represente a tela toda e que a partir dele se chamem os outros composables

## Componentes de UI Fundamentais

### Layout 
Componentes que visam o posicionamento dos elementos na tela

* Row
* Column
* Box
* BoxWithConstraints
* ConstraintLayout
<br/>

### Foundation 
Componentes de UI básicos

* Image
* Text
* BaseTextField
* LazyColumn
* LazyRow
* Shape
* entre outros
<br/>

#### Material
Componentes do material design (a maioria da UI vem daqui)

* AlertDialog
* Button
* Card
* CircularProgressIndicator
* DropdownMenu
* Checkbox
* FloatingActionButton
* LinearProgressIndicator
* ModalDrawer
* RadioButton
* Scaffold
* Slider
* Snackbar
* Switch
* TextField
* TopAppBar 
* BottomNavigation

> Para mais detalhes sobre esses composables e outros não incluídos na listagem, acesse esse [link][1], que te leva direto para uma página que contém diversos desses composables do Material e como utilizá-los. Caso queira mais detalhes sobre o Material como design system, acesse [Material 2][2] ou [Material 3][3]

## Detalhes de layout

**Row**: coloca os composables em uma linha, os adicionando no eixo horizontal da esquerda para a direita. Equivale ao *LinearLayout com orientação horizontal*
	
```kotlin
Row {
	// composables
}
```
<br/>

**Column**: coloca os composables em coluna, os adicionando no eixo vertical de cima para baixo. Equivale ao *LinearLayout com orientação vertical*
	
```kotlin
Column {
    // composables
}
```
<br/>

**Box**: alinha os composables empilhando-os num eixo de profundidade de trás pra frente, com os componentes mais recentes sobre os mais antigos na ordem de chamada (spoiler: dá pra modificar isso com o modificador *zIndex*). Equivale ao *FrameLayout*

```kotlin
Box {
	// composables
}
```
<br/>

**BoxWithConstraints**: mesma coisa que o *Box*, só que com esse composable é possível pegar dimensões como largura ou altura. O exemplo abaixo foi tirado de [Jetpack Compose Playground][4]

```kotlin
@Composable
fun BoxWithConstraintsDemo() {
    Column {
        Column {
            MyBoxWithConstraintsDemo()
        }

        Text("Here we set the size to 150.dp", modifier = Modifier.padding(top = 20.dp))
        Column(modifier = Modifier.size(150.dp)) {
            MyBoxWithConstraintsDemo()
        }
    }
}

@Composable
private fun MyBoxWithConstraintsDemo() {
    BoxWithConstraints {
        val boxWithConstraintsScope = this
        //You can use this scope to get the minWidth, maxWidth, minHeight, maxHeight in dp and constraints

        Column {
            if (boxWithConstraintsScope.maxHeight >= 200.dp) {
                Text(
                    "This is only visible when the maxHeight is >= 200.dp",
                    style = TextStyle(fontSize = 20.sp)
                )
            }
            Text("minHeight: ${boxWithConstraintsScope.minHeight}, maxHeight: ${boxWithConstraintsScope.maxHeight},  minWidth: ${boxWithConstraintsScope.minWidth} maxWidth: ${boxWithConstraintsScope.maxWidth}")
        }
    }
}
```
<br/>

**ConstraintLayout**: o sistema do compose que faz uso de recomposição é eficiente o suficiente pra que a gente não se preocupe com aninhamento de “views”, então é possível construir tudo de layout com *Rows*, *Columns* e *Boxes*. Mas caso a UI seja muito complexa. existe também a opção de usarmos o *ConstraintLayout*, este último só não oferece nenhuma vantagem de performance como no sistema de views anterior. 
    
> Obs: é necessário incluir uma lib específica do constraint layout compose no build.gradle do módulo: `implementation 'androidx.constraintlayout:constraintlayout-compose:1.0.1'`.

Abaixo temos um exemplo muito simples de alinhamento entre dois componentes onde o botão fica centralizado e abaixo da imagem
	
```kotlin
ConstraintLayout {
    val (image, button) = createRefs()
    Image(
        painter = painterResource(R.drawable.your_drawable)
        modifier = Modifier
            .constrainAs(image) {
                top.linkTo(parent.top)
                start.linkTo(parent.start)
            }
    )
    Button(
        onClick = {},
        modifier = Modifier
            .constrainAs(button) {
            	top.linkTo(image.bottom)
                start.linkTo(image.start)
                end.linkTo(image.end)
            }
    ) {
        Text("Click Me!")
    }
}
```
<br/>

> Aproveitando a deixa, para carregarmos imagens a partir de URL's, a Google indica duas libs: *Coil* ou *Glide*. A lib *Coil* é a mais frequentemente utilizada pela comunidade, inclusive em tutoriais. No *build.gradle* do módulo devemos inserir a seguinte dependência: `implementation("io.coil-kt:coil-compose:2.2.2")`.
> Para usarmos no compose, chamamos o composable *AsyncImage*, cujos principais parâmetros são *model*, que recebe a URL, e *contentDescription*, que pode ser um texto descrevendo o que temos na imagem.
> ```kotlin
> AsyncImage(
>     model = "url da imagem aqui",
>     contentDescription = "texto descritivo da imagem aqui"
> )
> ```


## Modifiers
Os modifiers são modificadores que vão determinar como um composable é desenhado. Todos os composables fundamentais possuem esse parâmetro.

```kotlin
Box(
    modifier = Modifier
        .modifier0()
        .modifier1()
        .modifier2()
        .modifier3()
        [...]
        .modifierN()
)
```
<br/>
Os modifiers são aplicados na ordem em que são definidos, e podem ser repetidos. Por exemplo, podemos aplicar o modifier de padding uma ou mais vezes na cadeia de modifiers, porém a ordem em que eles são definidos muda a forma que o composable é desenhado.

Dentre os modifiers, os mais usados são:
- **width**: define a largura do composable, se não for definido o tamanho se limita ao seu conteúdo.

- **height**: define a altura do composable, se não for definido o tamanho se limita ao seu conteúdo.

- **size**: com os parâmetros *width* e *height*, definem o tamanho do composable, se um deles for omitido, a dimensão se limita ao seu conteúdo

- **widthIn** e **heightIn**: são modifiers que limitam o composable a dimensões mínimas (setando o parâmetro *min*) e/ou máximas (setando o  parâmetro *max*)

- **fillMaxSize()**, **fillMaxWidth()** e **fillMaxHeight()**: o composable toma todo espaço disponível, toda largura ou toda altura, respectivamente.

- **padding**: determina um espaçamento pro composable. Pra setar um espaçamento em um ou mais cantos do componente, é necessário atribuir um valor ao seu respectivo parâmetro nomeado - e são eles: *top*, *start*, *end*, *bottom* e *all*. Se um parâmetro nomeado não for chamado, ele aplica o espaçamento definido em todos os cantos do composable, ou seja, é como se setasse o *all* implicitamente.

- **offset**: determina onde num eixo (x, y) o componente vai começar a ser desenhado. A principal diferença desse modificador para o padding é que o padding faz parte do componente, o espaçamento do offset não. Além disso, o offset só pode definir o espaçamento à esquerda, setando o *x*, e/ou o superior, setando o *y*.

```kotlin
Modifier
    .offset(x = 2.dp)
    .padding(
        top = 5.dp,
        start = 4.dp
    )
```

> Vale notar que por padrão não temos margens nos composables, então as formas de definirmos espaçamentos são ou com o modifier *padding* ou usando o composable *Spacer* (este pra definir espaçamento entre composables - é só um composable preenchedor de espaço). O modifier *offset* também é uma opção, dado seus limites.

- **zIndex**: é capaz de modificar a profundidade de um composable desenhado na tela, o trazendo pra frente de outros composables ou o levando pra trás.

- **border**: define a borda do composable, devendo passar a espessura, a cor e opcionalmente um *Shape*

- **clip**: determina o corte ou formato do composable a partir de um *Shape*. Por Exemplo, se passarmos um `RoundCornerShape(8.dp)` teremos um composable com bordas arredondadas de raio 8dp

- **background**: define o fundo do composable

- **then**: é um modificador especial que permite que chamemos outro modificador em cadeia - geralmente definido como parâmetro do composable - aplicando todos os modificadores definidos nesse modifier chamado. No exemplo abaixo, vemos que o *background* é aplicado após a definição do tamanho da *Column*.

```kotlin
@Composable
fun ExampleWithModifier(
    modifier: Modifier = Modifier
) {
    Column(
        modifier = Modifier
            .fillMaxWidth()
            .then(modifier)
    )
}

@Composable
fun ExampleCallingAboveComponent() {
    ExampleWithModifier(
        modifier = Modifier.background(Color.Blue)
    )
}
```
<br/>

## O que temos por vir
Ainda há muitos pontos a serem discutidos, como: manipulação de estado, animações, navegação, injeção de dependência, arquitetura, melhores práticas, etc, etc, etc. Mas só pra termos um exemplo de como é uma animação, temos aqui dois códigos extraídos do [Android Developers][5], demonstrando uma animação de valor e uma animação de visibilidade, respectivamente. A animação de valor está definindo a opacidade do composable de acordo com um condicional (**enabled**) e a animação de visibilidaade faz uso das transições `fadeIn()` e `fadeOut()` pra animar visibilidade dos fundos, transicionando entre as cores azul e cinza.

```kotlin
val alpha: Float by animateFloatAsState(if (enabled) 1f else 0.5f)
Box(
    Modifier.fillMaxSize()
        .graphicsLayer(alpha = alpha)
        .background(Color.Red)
)
```

```kotlin
AnimatedVisibility(
    visible = visible,
    enter = fadeIn(),
    exit = fadeOut()
) { // this: AnimatedVisibilityScope
    // Use AnimatedVisibilityScope#transition to add a custom animation
    // to the AnimatedVisibility.
    val background by transition.animateColor { state ->
        if (state == EnterExitState.Visible) Color.Blue else Color.Gray
    }
    Box(modifier = Modifier.size(128.dp).background(background))
}
```

  [1]: https://developer.android.com/reference/kotlin/androidx/compose/material3/package-summary#top-level-functions
  [2]: https://m2.material.io
  [3]: https://m3.material.io
  [4]: https://foso.github.io/Jetpack-Compose-Playground/foundation/layout/boxwithconstraints/
  [5]: https://developer.android.com/jetpack/compose/animation
