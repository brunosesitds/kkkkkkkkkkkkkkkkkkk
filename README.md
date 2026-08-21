export default function Diferenciais (){
    <section className='diferenciais' id='diferencias'>
        <div className='cards'>
            {listaDiferenciais.map((item) => (
                <div className='card' key={item.id}>
                    <img src={item.imagem} alt={item.alt} />
                    <p>{item.texto}</p>
                </div>
            ))}
        </div>
    </section>
}
