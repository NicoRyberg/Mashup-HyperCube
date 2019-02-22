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

<details><summary>Code Snippet</summary>
<p>
  
```JavaScript
app.getObjectProperties('skjU').then(function(model){ 
  app.createCube(model.properties.qHyperCubeDef, grafico_valor_cota);
});
```
</p></details>


O que está acontecendo neste snippet é:
•	O método getObjectProperties do objeto app retorna as propriedades do objeto com id = “ID DO GRÁFICO”.
•	Estas propriedades são enviadas como parâmetro para a função de callback, a qual é responsável por chamar o método createCube do objeto app.
•	Este método recebe dois parâmetros, um objeto de definição de hipercubo (explicado no início do documento) e uma função callback, a ser executada sempre que o hipercubo receber novos dados. Esta função recebe como parâmetro um hipercubo.


Para criar o gráfico, agora, pode ser usada qualquer biblioteca de gráfico do JS. Os dados que serão inseridos nesse gráfico, são retirados do hipercubo recebido na função call-back do método “createCube”.
Os dados se encontram nos campos “qMatrix” do array “qDataPages”. Segue snippet de código que trata os dados de um HiperCubo para serem inseridos em um gráfico combinado de barras e linha do CanvasJS:

<details><summary>Code Snippet</summary>
<p>

```JavaScript
function grafico_valor_cota(reply){
  valor_data_dict = new Array();
  cota_data_dict = new Array();

  $.each(reply.qHyperCube.qDataPages[0].qMatrix, function (key, value){
    X = value[0]['qNum'];
    X_label = value[0]['qText'];
    Y_v = Math.round(value[1]['qNum']);
    Y_c = Math.round(value[2]['qNum']);

    valor_data_dict.push({'x':X,'y':Y_v, 'label':X_label});
    cota_data_dict.push({'x':X,'y':Y_c, 'label':X_label});
  });

  width = $("#jsnative_chart_valorXcota").width();
  chart = new CanvasJS.Chart("jsnative_chart_valorXcota",
  {
    title:{
    text: "Valor X Cota",
      fontColor: "rgb(128, 128, 128)",
    fontSize: 16,
    horizontalAlign: "left",
    marginBottom: 100,
    },   
    data: [{
      name: "Valor",
    type: "column",
    color:"#0F4DBC",
    dataPoints: valor_data_dict
    },
    {       
      name: "Cota",
    type: "line",
    color:"black",
    markerSize: 10,
    dataPoints: cota_data_dict
    }],
    backgroundColor: "transparent",
    zoomEnabled:true,
    zoomType: "x",
    axisY:{
      gridColor: "transparent",
    tickLength: 0,
    lineThickness:0,
    margin:0,
    labelFontSize: 14,
    labelFormatter: function (e){
      val = e.value.toString()
      if (val.length > 3){
        var dots = new Array();
        for(let j=3; j<val.length; j=j+3){
          dots.push(j);						
        }
        for(let j=0; j<dots.length; j++){
          pos = val.length - dots[j] - j
          val = val.slice(0, pos) + "." + val.slice(pos);
        }
      }
      return val
    }
    },
    axisX:{
      labelFontSize: 14,
      },
    height: 500,
    width: width,
    toolTip: {
    shared: true,
    contentFormatter: function (e) {
      var content = "Dia: <strong>"+ e.entries[0].dataPoint.label +"</strong><br/><br/>";
      for (var i = 0; i < e.entries.length; i++) {
        val = e.entries[i].dataPoint.y.toString()
        if (val.length > 3){
          var dots = new Array();
          for(let j=3; j<val.length; j=j+3){
            dots.push(j);						
          }
          for(let j=0; j<dots.length; j++){
            pos = val.length - dots[j] - j
            val = val.slice(0, pos) + "." + val.slice(pos);
          }
        }
        content += e.entries[i].dataSeries.name + ": <strong> R$ " + val + " mil</strong>";
        content += "<br/>";
      }
      return content;
    }
    },
  });

  chart.render();
}
```
</p></details>
  
  
O que está acontecendo neste snippet é, itera-se sobre os elementos contidos no qMatrix para gerar um dicionário contendo os dados, no formato aceito pelo construtor do CanvasJS. (Como o gráfico sendo utilizado possui somente um dataPage, é acessado o primeiro elemento do Array).
