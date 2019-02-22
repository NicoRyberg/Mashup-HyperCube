# Mashup-HyperCube
Qlik Chart to Native JS Chart, using predefined HyperCube

## HyperCubes

Todos os gráficos no Qlik têm um HyperCube atribuído. Este hipercubo tem as definições por padrão do Qlik, com base no tipo de gráfico criado. Por exemplo, algumas das definições são:
•	Dimensões e measures do gráfico
•	Condições de cálculo, ou seja, que deve ser cumprida para que os dados do hipercubo sejam (re)calculados, enquanto essas propriedades não sejam atendidas, a engine não executa um novo cálculo.
•	Link com as propriedades de definição de um hipercubo: https://help.qlik.com/en-US/sense-developer/November2018/apis/EngineAPI/definitions-HyperCubeDef.html

Com estas definições, pode ser criado um objeto de hipercubo, o qual é responsável por armazenar os dados e metadados do gráfico. Estes dados serão recalculados com base nas condições de cálculo definidas, por exemplo, caso não se tenham restrições de cálculo, a cada nova seleção, os dados são recalculados. Os dados de um hipercubo encontram-se na propriedade “qDataPages”.

## Tratando um HyperCube na prática

Para simplificação, criaremos um hipercubo com as definições de hipercubo geradas para um gráfico do Qlik. Para pegar este objeto de definição de hipercubo, e criar um novo hipercubo com as mesmas definições, usa-se o seguinte código:

[Create HyperCube](https://raw.github.com/NicoRyberg/Mashup-HyperCube//master/imgs/createhypercube.png)

O que está acontecendo neste snippet é:
•	O método getObjectProperties do objeto app retorna as propriedades do objeto com id = “ID DO GRÁFICO”.
•	Estas propriedades são enviadas como parâmetro para a função de callback, a qual é responsável por chamar o método createCube do objeto app.
•	Este método recebe dois parâmetros, um objeto de definição de hipercubo (explicado no início do documento) e uma função callback, a ser executada sempre que o hipercubo receber novos dados. Esta função recebe como parâmetro um hipercubo.


Para criar o gráfico, agora, pode ser usada qualquer biblioteca de gráfico do JS. Os dados que serão inseridos nesse gráfico, são retirados do hipercubo recebido na função call-back do método “createCube”.
Os dados se encontram nos campos “qMatrix” do array “qDataPages”. Segue snippet de código que trata os dados de um HiperCubo para serem inseridos em um gráfico combinado de barras e linha do CanvasJS:

[HyperCube Callback](https://raw.github.com/NicoRyberg/Mashup-HyperCube//master/imgs/hypercubecallback.png)

O que está acontecendo neste snippet é, itera-se sobre os elementos contidos no qMatrix para gerar um dicionário contendo os dados, no formato aceito pelo construtor do CanvasJS. (Como o gráfico sendo utilizado possui somente um dataPage, é acessado o primeiro elemento do Array).
