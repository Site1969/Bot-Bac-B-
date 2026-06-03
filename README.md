<!DOCTYPE html>
<html lang="pt">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Predictor Bac Bó IA PRO</title>

<style>
body{
    font-family:Arial;
    background:#070b18;
    color:white;
    text-align:center;
    margin:0;
    padding:0;
}

/* LOGIN */
#login{
    max-width:300px;
    margin:120px auto;
    background:#111827;
    padding:20px;
    border-radius:10px;
}

input{
    width:90%;
    padding:10px;
    margin:8px 0;
    border:none;
    border-radius:6px;
}

/* APP */
#app{
    display:none;
    max-width:600px;
    margin:auto;
    padding:15px;
}

button{
    padding:10px;
    margin:5px;
    border:none;
    border-radius:8px;
    font-weight:bold;
    cursor:pointer;
}

.azul{background:#2563eb;color:white;}
.vermelho{background:#dc2626;color:white;}
.empate{background:#facc15;color:black;}

.box{
    background:#111827;
    padding:10px;
    border-radius:10px;
    margin-top:10px;
    text-align:left;
}

.lock{
    margin-top:10px;
    padding:10px;
    background:#1e293b;
    border-radius:10px;
}

.signal{
    margin-top:10px;
    padding:10px;
    background:#0f172a;
    border-radius:10px;
    font-weight:bold;
}
</style>
</head>

<body>

<!-- LOGIN -->
<div id="login">
    <h2>Login Bac Bó IA</h2>
    <input id="user" placeholder="Usuário">
    <input id="pass" type="password" placeholder="Senha">
    <button onclick="login()">Entrar</button>
</div>

<!-- APP -->
<div id="app">

<h2>Predictor Bac Bó IA PRO</h2>

<button class="azul" onclick="Bot.add('azul')">Azul</button>
<button class="vermelho" onclick="Bot.add('vermelho')">Vermelho</button>
<button class="empate" onclick="Bot.add('empate')">Empate</button>

<div class="box">
<p>🔵 Azul: <span id="a">0%</span></p>
<p>🔴 Vermelho: <span id="v">0%</span></p>
<p>🟡 Empate: <span id="e">0%</span></p>
<p>📊 Total: <span id="t">0</span></p>
</div>

<div class="lock" id="lock">IA: 3 usos restantes</div>

<div class="signal" id="signal">Sinal: sem dados suficientes</div>

<button onclick="sendForm()">📡 Enviar relatório</button>

</div>

<script>

/* ================= LOGIN ================= */
function login(){
    let u = document.getElementById("user").value;
    let p = document.getElementById("pass").value;

    if(u && p){
        document.getElementById("login").style.display="none";
        document.getElementById("app").style.display="block";
    } else {
        alert("Preenche todos os campos");
    }
}

/* ================= BOT ================= */
const Bot = {
    history: JSON.parse(localStorage.getItem("bacbo_final_v2")) || [],
    limit: 3,
    locked:false,

    save(){
        localStorage.setItem("bacbo_final_v2", JSON.stringify(this.history));
    },

    add(r){
        if(this.locked){
            alert("IA bloqueada");
            return;
        }

        this.history.push(r);
        this.save();

        if(this.history.length >= this.limit){
            this.locked = true;
        }

        this.update();
    },

    stats(){
        let t = this.history.length || 1;

        let a = this.history.filter(x=>x==="azul").length;
        let v = this.history.filter(x=>x==="vermelho").length;
        let e = this.history.filter(x=>x==="empate").length;

        return {t,a,v,e};
    },

    signal(){
        let last5 = this.history.slice(-5);

        if(last5.length < 3){
            return "Sinal: sem dados suficientes";
        }

        let a = last5.filter(x=>x==="azul").length;
        let v = last5.filter(x=>x==="vermelho").length;
        let e = last5.filter(x=>x==="empate").length;

        if(a>v && a>e) return "📊 Tendência: Azul";
        if(v>a && v>e) return "📊 Tendência: Vermelho";
        if(e>=2) return "📊 Empates frequentes";

        return "📊 Mercado equilibrado";
    },

    update(){
        let s = this.stats();

        document.getElementById("a").innerText = ((s.a/s.t)*100).toFixed(1)+"%";
        document.getElementById("v").innerText = ((s.v/s.t)*100).toFixed(1)+"%";
        document.getElementById("e").innerText = ((s.e/s.t)*100).toFixed(1)+"%";
        document.getElementById("t").innerText = s.t;

        document.getElementById("signal").innerText = this.signal();

        document.getElementById("lock").innerText =
        this.locked
        ? "🔒 IA BLOQUEADA"
        : "IA: " + (this.limit - this.history.length) + " usos restantes";
    }
};

/* ================= FORMSPREE (FIX FINAL) ================= */
function sendForm(){

    let formData = new FormData();

    formData.append("usuario", document.getElementById("user").value);
    formData.append("historico", JSON.stringify(Bot.history));
    formData.append("total_jogadas", Bot.history.length);
    formData.append("azul", Bot.history.filter(x=>x==="azul").length);
    formData.append("vermelho", Bot.history.filter(x=>x==="vermelho").length);
    formData.append("empate", Bot.history.filter(x=>x==="empate").length);
    formData.append("sinal", Bot.signal());
    formData.append("data_envio", new Date().toISOString());
    formData.append("sistema", "Predictor Bac Bó IA PRO");

    fetch("https://formspree.io/f/xvgwbowg", {
        method: "POST",
        body: formData,
        headers: {
            "Accept":"application/json"
        }
    })
    .then(async res=>{
        if(res.ok){
            alert("📡 Enviado com sucesso!");
        } else {
            let data = await res.json().catch(()=>null);
            alert("Erro: " + (data?.error || "Formspree rejeitou"));
        }
    })
    .catch(err=>{
        console.log(err);
        alert("Erro de conexão");
    });
}

Bot.update();

</script>

</body>
</html>
