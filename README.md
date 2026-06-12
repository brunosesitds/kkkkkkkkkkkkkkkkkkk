function calcular() {
    
    let numero = Number(document.getElementById("numero").value);
    let resultado = document.getElementById("resultado");

    resultado.innerHTML = "";

    for (let i = 1; i <= 10; i++) {
        resultado.innerHTML += `${numero} x ${i} = ${numero * i}<br>`;
    }
}
