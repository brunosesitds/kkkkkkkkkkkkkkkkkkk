@import url('https://fonts.googleapis.com/css2?family=Jacques+Francois&display=swap');

*{
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

html, body{
    max-width: 100%;
    overflow-x: hidden;
}

body{
    background: #000;
    color: #fff;
}

.header{
    position: absolute;
    width: 100%;
    top: 0;
    left: 0;
    background-color: #1c1c1c;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 20px 80px;
    z-index: 1000;
}

.logo{
    font-size: 28px;
    font-weight: 900;
}

.menu-toggle{
    display: flex;
    flex-direction: column;
    cursor: pointer;
    z-index: 1100;
}

.menu-toggle span{
    width: 25px;
    height: 3px;
    margin: 4px 0;
    transition: 0.3;
    background: white;
}

.menu{
    position: fixed;
    top: 0;
    right: 0;
    width: 40%;
    height: 100vh;
    background-color: #1c1c1c;
    display: flex;
    flex-direction: column;
    padding: 80px 40px;
    transform: translateX(100%);
    transition: transform 0.4s ease;

}

.menu a{
    margin: 15px 0;
    text-decoration: none;
    font-size: 18px;
    color: #fff;
}
