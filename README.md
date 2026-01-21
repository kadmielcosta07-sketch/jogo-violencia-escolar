# jogo-violencia-escolar
Um jogo sobre temas de violência na escola, onde o aluno responde com o verdadeiro ou falso. Sendo um recursos de mentoria e tratamentos de utilização para as turmas desafiadoras!

<!DOCTYPE html>
<html lang='pt-BR'>
<head>
<meta charset='UTF-8'>
<meta name='viewport' content='width=device-width, initial-scale=1'>
<title>Jogo: Decisões Contra a Violência Escolar</title>
<style>
  :root{
    --bg:#000; --card:#171717; --muted:#9aa0a6; --ok:#15c87a; --bad:#ff5a5a; --accent:#ffd60a;
  }
  *{box-sizing:border-box}
  body{margin:0;background:var(--bg);color:#fff;font-family:Arial,Helvetica,sans-serif;}
  header{padding:24px 16px;text-align:center;border-bottom:1px solid #222}
  h1{margin:0;color:var(--accent);font-size:clamp(22px,4vw,34px)}
  #app{max-width:900px;margin:0 auto;padding:24px 16px}
  .status{display:flex;gap:12px;flex-wrap:wrap;align-items:center;justify-content:space-between;margin-bottom:16px}
  .pill{background:#111;border:1px solid #222;color:#ddd;border-radius:999px;padding:8px 12px;font-size:14px}
  .bar{position:relative;height:10px;background:#111;border:1px solid #222;border-radius:999px;overflow:hidden}
  .bar>span{position:absolute;left:0;top:0;bottom:0;background:linear-gradient(90deg,#ffd60a,#ff9f1a);width:0%}
  .card{background:var(--card);border:1px solid #222;border-radius:12px;padding:20px}
  .enunciado{font-size:18px;line-height:1.5}
  .actions{margin-top:18px}
  button{appearance:none;border:1px solid #333;background:#2a2a2a;color:#fff;border-radius:10px;padding:10px 16px;cursor:pointer}
  button:hover{filter:brightness(1.1)}
  .btn-primary{background:#3a3a3a}
  .btn-next{background:#ffd60a;color:#121212;border-color:#ffd60a;font-weight:700}
  .feedback{margin-top:14px;padding:12px;border-left:6px solid var(--accent);background:#111;border-radius:8px}
  .feedback.ok{border-left-color:var(--ok)}
  .feedback.bad{border-left-color:var(--bad)}
  .choices{display:flex;gap:10px;flex-wrap:wrap}
  .score-ok{color:var(--ok);font-weight:700}
  .score-bad{color:var(--bad);font-weight:700}
  .meta{color:#bbb;font-size:14px;margin-top:6px}
  footer{opacity:.8;text-align:center;padding:24px 16px;color:var(--muted)}
</style>
</head>
<body>
  <header>
    <h1>Jogo: Decisões Contra a Violência Escolar</h1>
  </header>
  <div id="app">
    <div class="status">
      <div class="pill">Pergunta <span id="qnum">1</span>/<span id="qtotal">--</span></div>
      <div class="pill">Pontuação: <span id="score">0</span></div>
      <div class="pill">Melhor sequência: <span id="best">0</span></div>
      <div class="pill" style="flex:1 1 220px;min-width:220px">
        <div class="bar"><span id="progress" style="width:0%"></span></div>
      </div>
    </div>

    <div class="card">
      <div id="enunciado" class="enunciado"></div>
      <div class="actions">
        <div class="choices">
          <button id="btnV" class="btn-primary">Verdadeiro</button>
          <button id="btnF" class="btn-primary">Falso</button>
        </div>
        <div id="feedback" class="feedback" style="display:none"></div>
        <div id="meta" class="meta" style="display:none"></div>
        <div style="margin-top:12px">
          <button id="btnNext" class="btn-next" style="display:none">Próxima</button>
        </div>
      </div>
    </div>
  </div>
  <footer>
    Sem temporizador. Game de tomada de decisão educativa.
  </footer>

<script>
const questions = [
  {s:'Um aluno empurra outro no corredor. A solução: "Ignorar a situação porque é brincadeira".', c:false, f:'Ignorar comportamentos agressivos reforça a violência. Busque ajuda de um adulto responsável imediatamente.'},
  {s:'Um estudante sofre xingamentos constantes. A solução: "Conversar com um adulto responsável da escola".', c:true, f:'Buscar apoio de adultos é uma das formas mais eficazes de combater a violência escolar.'},
  {s:'Dois alunos brigam na quadra. A solução: "Separá-los fisicamente sem ajuda".', c:false, f:'Intervir fisicamente é perigoso. Procure um funcionário da escola ou responsável pelo pátio.'},
  {s:'Colegas espalham boatos sobre um aluno. A solução: "Não repassar as informações e avisar a coordenação".', c:true, f:'Boatos são violência psicológica. Não reforçar e comunicar é essencial.'},
  {s:'Um aluno ameaça outro. A solução: "Registrar a ameaça e informar a direção".', c:true, f:'Ameaças devem ser documentadas e reportadas para evitar escalada.'},
  {s:'Um estudante é isolado pelos colegas. A solução: "Dizer que ele deve se acostumar".', c:false, f:'Isolamento social é uma forma de violência. A escola deve intervir para inclusão.'},
  {s:'Alunos escondem materiais escolares de outro. A solução: "Conversar com os envolvidos e mediar o conflito".', c:true, f:'A mediação ajuda a resolver conflitos e restaurar relações.'},
  {s:'Violência verbal em sala. A solução: "Responder com mais agressividade".', c:false, f:'Responder com agressão amplia o conflito. Busque apoio de um adulto.'},
  {s:'Humilhação pública de um aluno. A solução: "Informar imediatamente um professor".', c:true, f:'Humilhação é grave e exige intervenção imediata.'},
  {s:'Aluno intimida outro para pegar dinheiro. A solução: "Entregar o dinheiro para evitar problemas".', c:false, f:'Ceder reforça o agressor. Notifique a escola e responsáveis.'},
  {s:'Violência durante o recreio. A solução: "Registrar e procurar a coordenação".', c:true, f:'Registros ajudam a prevenir novos casos e orientar ações.'},
  {s:'Aluno recebe mensagens ofensivas online. A solução: "Bloquear e denunciar".', c:true, f:'Cyberbullying deve ser denunciado aos canais da escola e plataformas.'},
  {s:'Um colega faz piadas ofensivas. A solução: "Rir para evitar conflitos".', c:false, f:'Rir reforça a agressão. Mostre que não é aceitável e busque apoio.'},
  {s:'Agressão no banheiro. A solução: "Intervir sozinho".', c:false, f:'Intervir sozinho é arriscado. Chame um adulto responsável.'},
  {s:'Aluno chora após ofensas. A solução: "Confortar e chamar ajuda".', c:true, f:'Apoiar a vítima reduz danos emocionais e encoraja relatos.'},
  {s:'Colegas empurram aluno em fila. A solução: "Dizer para parar e buscar adulto".', c:true, f:'Intervenções seguras e pedido de apoio ajudam a conter.'},
  {s:'Aluno grava briga para postar. A solução: "Compartilhar o vídeo".', c:false, f:'Filmar/compartilhar violência é inadequado e pode ser crime.'},
  {s:'Aluno sofre extorsão. A solução: "Contar aos pais e à escola".', c:true, f:'Apoio familiar e escolar é essencial em casos de extorsão.'},
  {s:'Aluno diz que vai se vingar fisicamente. A solução: "Incentivar a briga".', c:false, f:'Violência gera mais violência. Promova mediação e diálogo.'},
  {s:'Brincadeira que machuca. A solução: "Conversar sobre limites e pedir supervisão".', c:true, f:'Brincadeiras perigosas precisam de limites e supervisão.'}
];

// Estado
let idx = 0;            // índice da pergunta atual
let score = 0;          // pontuação total
let streak = 0;         // sequência atual de acertos
let bestStreak = 0;     // melhor sequência

const qnum = document.getElementById('qnum');
const qtotal = document.getElementById('qtotal');
const progress = document.getElementById('progress');
const enunciado = document.getElementById('enunciado');
const btnV = document.getElementById('btnV');
const btnF = document.getElementById('btnF');
const btnNext = document.getElementById('btnNext');
const feedback = document.getElementById('feedback');
const meta = document.getElementById('meta');
const scoreEl = document.getElementById('score');
const bestEl = document.getElementById('best');

qtotal.textContent = questions.length;

function render(){
  qnum.textContent = idx + 1;
  const pct = (idx)/questions.length*100;
  progress.style.width = pct + '%';
  const q = questions[idx];
  enunciado.textContent = `${idx+1}. ${q.s}`;
  feedback.style.display = 'none';
  feedback.className = 'feedback';
  meta.style.display = 'none';
  meta.textContent = '';
  btnNext.style.display = 'none';
  btnV.disabled = false; btnF.disabled = false;
}

function finish(){
  const pct = Math.round(score/(questions.length*4)*100); // pontuação máxima por questão é 4 (1 base + 3 bônus)
  document.querySelector('.card').innerHTML = `
    <h2 style="color: var(--accent); margin-top:0">Resultado Final</h2>
    <p style="font-size:18px">Pontuação total: <span class="score-ok">${score}</span></p>
    <p style="font-size:18px">Melhor sequência de acertos: <span class="score-ok">${bestStreak}</span></p>
    <p style="color:#ddd">Aproveitamento considerando o bônus de sequência: ${isNaN(pct)?0:pct}%</p>
    <div style="margin-top:16px; display:flex; gap:10px; flex-wrap:wrap">
      <button onclick="location.reload()">Jogar novamente</button>
    </div>
  `;
  progress.style.width = '100%';
}

function answer(userChoice){
  const q = questions[idx];
  const correct = q.c === userChoice;

  let gained = 0;
  if (correct){
    streak += 1;
    if (streak > bestStreak) bestStreak = streak;
    // Regra D: +1 por acerto + bônus por sequência (+1 extra por acerto consecutivo, limitado a +3)
    // Ex.: 1º acerto = +1; 2º consecutivo = +2; 3º = +3; 4º+ = +4 (teto +3 extra)
    const bonus = Math.min(streak - 1, 3);
    gained = 1 + bonus;
    score += gained;
  } else {
    // errou: zera a sequência
    streak = 0;
  }

  // Atualiza cabeçalho
  scoreEl.textContent = score;
  bestEl.textContent = bestStreak;

  // Feedback colorido
  feedback.style.display = 'block';
  feedback.textContent = (correct ? 'Correto! ' : 'Resposta incorreta. ') + q.f;
  feedback.className = 'feedback ' + (correct ? 'ok' : 'bad');

  // Meta info (quanto ganhou nesta questão e sequência atual)
  meta.style.display = 'block';
  meta.innerHTML = correct
    ? `+${gained} ponto(s) nesta questão • sequência atual: ${streak}`
    : `0 ponto(s) nesta questão • sequência encerrada.`;

  // Desabilita botões até ir para próxima
  btnV.disabled = true; btnF.disabled = true;
  btnNext.style.display = 'inline-block';
}

btnV.addEventListener('click', ()=>answer(true));
btnF.addEventListener('click', ()=>answer(false));
btnNext.addEventListener('click', ()=>{
  if (idx < questions.length - 1){
    idx++;
    render();
  } else {
    finish();
  }
});

render(); 
</script>
</body>
</html>
