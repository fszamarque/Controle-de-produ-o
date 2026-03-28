# Controle-de-produ-o
Sistema de controle de produção industrial com análise de refugo e dashboard
<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Controle de Produção MASTER</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body { font-family: Arial; background: #eef2f5; padding: 10px; }
h1 { text-align: center; }
.container { max-width: 1100px; margin: auto; background: white; padding: 15px; border-radius: 12px; box-shadow: 0 0 15px rgba(0,0,0,0.1);} 
input, select, button { width: 100%; padding: 10px; margin: 5px 0; }
button { background: #007bff; color: white; border: none; cursor: pointer; }
button:hover { background: #0056b3; }
table { width: 100%; margin-top: 20px; border-collapse: collapse; }
th, td { padding: 8px; border: 1px solid #ddd; text-align: center; font-size: 14px; }
.ok { background-color: #c8f7c5; }
atraso { background-color: #f7c5c5; }
canvas { margin-top: 30px; }
@media(max-width:600px){ body{padding:5px;} }
</style>
</head>
<body>

<div class="container" id="loginTela">
<h2>Login do Operador</h2>
<input type="text" id="operador" placeholder="Digite seu nome">
<button onclick="login()">Entrar</button>
</div>

<div class="container" id="sistema" style="display:none;">
<h1>Controle de Produção MASTER</h1>

<p><b>Operador:</b> <span id="nomeOperador"></span></p>

<select id="maquina">
<option value="">Máquina</option>
<option>Laser</option>
<option>Dobra</option>
<option>Punção</option>
</select>

<input type="text" id="peca" placeholder="Código da Peça">
<input type="number" id="producao" placeholder="Produção">
<input type="number" id="refugo" placeholder="Refugo">
<input type="number" id="meta" placeholder="Meta do turno">

<button onclick="salvar()">Salvar</button>
<button onclick="exportarCSV()">Exportar Excel</button>
<button onclick="limpar()">Limpar Dados</button>

<table>
<thead>
<tr>
<th>Operador</th>
<th>Máquina</th>
<th>Peça</th>
<th>Produção</th>
<th>Refugo</th>
<th>Meta</th>
<th>Status</th>
</tr>
</thead>
<tbody id="tabela"></tbody>
</table>

<canvas id="grafico"></canvas>
</div>

<script>
let dados = JSON.parse(localStorage.getItem('dados')) || [];
let operadorLogado = '';
let chart;

function login(){
  let nome = document.getElementById('operador').value;
  if(!nome) return alert('Digite seu nome');
  operadorLogado = nome;
  document.getElementById('nomeOperador').innerText = nome;
  document.getElementById('loginTela').style.display = 'none';
  document.getElementById('sistema').style.display = 'block';
}

function salvar(){
  let maquina = document.getElementById('maquina').value;
  let peca = document.getElementById('peca').value;
  let producao = parseInt(document.getElementById('producao').value);
  let refugo = parseInt(document.getElementById('refugo').value) || 0;
  let meta = parseInt(document.getElementById('meta').value) || 0;

  if(!maquina || !peca || !producao) return alert('Preencha tudo');

  let status = (refugo > producao*0.1 || producao < meta) ? 'ATENÇÃO' : 'OK';

  dados.push({operador: operadorLogado, maquina, peca, producao, refugo, meta, status});
  localStorage.setItem('dados', JSON.stringify(dados));

  atualizarTabela();
  atualizarGrafico();
}

function atualizarTabela(){
  let tabela = document.getElementById('tabela');
  tabela.innerHTML='';

  dados.forEach(d=>{
    tabela.innerHTML += `<tr class="${d.status==='OK'?'ok':'atraso'}">
    <td>${d.operador}</td>
    <td>${d.maquina}</td>
    <td>${d.peca}</td>
    <td>${d.producao}</td>
    <td>${d.refugo}</td>
    <td>${d.meta}</td>
    <td>${d.status}</td>
    </tr>`;
  });
}

function atualizarGrafico(){
  let maquinas={};

  dados.forEach(d=>{
    maquinas[d.maquina]=(maquinas[d.maquina]||0)+d.producao;
  });

  let labels=Object.keys(maquinas);
  let valores=Object.values(maquinas);

  if(chart) chart.destroy();

  chart = new Chart(document.getElementById('grafico'),{
    type:'bar',
    data:{ labels:labels, datasets:[{label:'Produção', data:valores}] }
  });
}

function exportarCSV(){
  let csv='Operador,Maquina,Peca,Producao,Refugo,Meta,Status\n';
  dados.forEach(d=>{
    csv+=`${d.operador},${d.maquina},${d.peca},${d.producao},${d.refugo},${d.meta},${d.status}\n`;
  });

  let blob=new Blob([csv],{type:'text/csv'});
  let link=document.createElement('a');
  link.href=URL.createObjectURL(blob);
  link.download='producao.csv';
  link.click();
}

function limpar(){
  localStorage.removeItem('dados');
  dados=[];
  atualizarTabela();
  atualizarGrafico();
}

atualizarTabela();
atualizarGrafico();
</script>

</body>
</html>
