function login(){
    var nome = $("#nome").val()
    var senha = $("#senha").val()

    if(nome && senha && nome === "admin" && senha === "12345"){
        const user = {
            name: nome,
            dataEntrada: new Date(),
            id: Math.floor(Math.random() * 1000000)
        }
        localStorage.setItem("usuario", JSON.stringify(user))
        window.location.href = "../loja"

    }else{
        document.getElementById("errorModal").style.display = "flex"
        document.getElementById("nome").style.borderBottom = "3px solid red"
        document.getElementById("senha").style.borderBottom = "3px solid red"
    }
}






function showPassword(){
    var inputSenha = document.querySelector("#senha")
    var img_eye = document.querySelector("#eye")

    if(inputSenha.getAttribute("type") === "password"){
        inputSenha.setAttribute("type", "text")
        img_eye.setAttribute("src", "../../public/hide.png")
    }else{
        inputSenha.setAttribute("type", "password")
        img_eye.setAttribute("src", "../../public/view.png")
    }
}
