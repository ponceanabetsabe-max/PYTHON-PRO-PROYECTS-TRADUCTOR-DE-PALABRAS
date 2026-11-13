import discord
from discord.ext import commands
import random
import os
import asyncio
from ddgs import DDGS

# ============================================
# 1. CONFIGURACION DEL BOT
# ============================================

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(
    command_prefix="&",
    intents=intents
)

# Ruta de las imagenes
BASE_DIR = os.path.dirname(os.path.abspath(__file__))
IMAGE_DIR = os.path.join(BASE_DIR, "images")


# ============================================
# 2. BASE DE DATOS DE EVIDENCIAS
# ============================================

evidencias = [
    {
        "fuente": "UNAM",
        "evidencia": "La contaminacion del aire puede afectar la salud respiratoria y el medio ambiente.",
        "recomendacion": "Evita realizar actividades al aire libre cuando la calidad del aire sea mala.",
        "enlace": "https://www.unam.mx/"
    },
    {
        "fuente": "IPN",
        "evidencia": "El IPN desarrolla investigaciones relacionadas con la contaminacion, el agua y los residuos.",
        "recomendacion": "Separa la basura y reduce el uso de plasticos.",
        "enlace": "https://www.ipn.mx/"
    },
    {
        "fuente": "UAM",
        "evidencia": "La investigacion ambiental permite estudiar los efectos de los contaminantes en los ecosistemas.",
        "recomendacion": "Evita arrojar basura y sustancias contaminantes en rios y mares.",
        "enlace": "https://www.uam.mx/"
    }
]


# ============================================
# 3. COMANDO ESTUDIO
# ============================================

@bot.command()
async def estudio(ctx):

    dato = random.choice(evidencias)

    mensaje = (
        "🌎 **DATO AMBIENTAL ALEATORIO**\n\n"
        f"🏫 **Fuente:** {dato['fuente']}\n\n"
        f"🔎 **Evidencia:** {dato['evidencia']}\n\n"
        f"💡 **Recomendacion:** {dato['recomendacion']}\n\n"
        f"📚 **Consulta:** {dato['enlace']}"
    )

    await ctx.send(mensaje)


# ============================================
# 4. COMANDO FUENTES
# ============================================

@bot.command()
async def fuentes(ctx):

    mensaje = (
        "📚 **FUENTES DE INFORMACION**\n\n"
        "🏛️ **UNAM**\n"
        "Universidad Nacional Autonoma de Mexico\n"
        "https://www.unam.mx/\n\n"
        "🏛️ **IPN**\n"
        "Instituto Politecnico Nacional\n"
        "https://www.ipn.mx/\n\n"
        "🏛️ **UAM**\n"
        "Universidad Autonoma Metropolitana\n"
        "https://www.uam.mx/\n\n"
        "🌱 Estas instituciones cuentan con investigaciones "
        "y recursos relacionados con temas ambientales."
    )

    await ctx.send(mensaje)


# ============================================
# 5. FUNCION DE BUSQUEDA EN INTERNET
# ============================================

def buscar_informacion(consulta, dominio):

    resultados = []

    busqueda = f"site:{dominio} {consulta}"

    try:

        with DDGS() as buscador:

            resultados = list(
                buscador.text(
                    busqueda,
                    max_results=2
                )
            )

    except Exception as error:

        print(f"Error al buscar en {dominio}: {error}")

    return resultados


# ============================================
# 6. COMANDO BUSCAR
# ============================================

@bot.command()
async def buscar(ctx, *, consulta=None):

    if not consulta:

        await ctx.send(
            "❌ Escribe el tema que deseas investigar.\n"
            "Ejemplo: `&buscar contaminacion del agua`"
        )
        return

    mensaje = (
        f"🔎 **BUSQUEDA DE INFORMACION**\n"
        f"🌎 Tema: **{consulta}**\n\n"
        "Estoy consultando las fuentes institucionales..."
    )

    aviso = await ctx.send(mensaje)

    instituciones = [
        ("UNAM", "unam.mx"),
        ("IPN", "ipn.mx"),
        ("UAM", "uam.mx")
    ]

    respuesta = (
        f"🌎 **RESULTADOS DE LA BUSQUEDA**\n"
        f"🔍 Tema: {consulta}\n\n"
    )

    encontrados = 0

    for nombre, dominio in instituciones:

        resultados = await asyncio.to_thread(
            buscar_informacion,
            consulta,
            dominio
        )

        respuesta += f"🏛️ **{nombre}**\n"

        if resultados:

            for resultado in resultados:

                titulo = resultado.get(
                    "title",
                    "Resultado sin titulo"
                )

                enlace = resultado.get(
                    "href",
                    ""
                )

                descripcion = resultado.get(
                    "body",
                    "Sin descripcion disponible."
                )

                if enlace:

                    respuesta += (
                        f"📌 **{titulo}**\n"
                        f"{descripcion[:300]}\n"
                        f"🔗 {enlace}\n\n"
                    )

                    encontrados += 1

        else:

            respuesta += (
                "No se encontraron resultados disponibles.\n\n"
            )

    respuesta += (
        "⚠️ Los resultados proceden de una busqueda web "
        "por dominio institucional. Revisa cada publicacion "
        "para comprobar su fecha, metodologia y contenido."
    )

    if encontrados == 0:

        respuesta = (
            "❌ No se encontraron resultados.\n"
            "Intenta con otras palabras o un tema diferente."
        )

    # Discord permite hasta 2000 caracteres por mensaje
    partes = [
        respuesta[i:i + 1900]
        for i in range(0, len(respuesta), 1900)
    ]

    await aviso.edit(content=partes[0])

    for parte in partes[1:]:

        await ctx.send(parte)


# ============================================
# 7. COMANDO MEME 1
# ============================================

@bot.command()
async def meme1(ctx):

    ruta = os.path.join(
        IMAGE_DIR,
        "meme1.jpeg"
    )

    if os.path.isfile(ruta):

        archivo = discord.File(
            ruta,
            filename="meme1.jpeg"
        )

        await ctx.send(
            "**ERROR EN LA LINEA 42**",
            file=archivo
        )

    else:

        await ctx.send(
            "❌ No se encontro meme1.jpeg. "
            "Revisa la carpeta images."
        )


# ============================================
# 8. COMANDO REFERENCIA
# ============================================

@bot.command()
async def ayuda(ctx):

    mensaje = (
        "🤖 **BOT DE CONTAMINACION AMBIENTAL**\n\n"

        "🌎 **COMANDOS DISPONIBLES**\n\n"

        "📌 `&estudio`\n"
        "Muestra una evidencia ambiental aleatoria.\n\n"

        "📌 `&fuentes`\n"
        "Muestra las fuentes institucionales.\n\n"

        "📌 `&buscar [tema]`\n"
        "Busca informacion en paginas de UNAM, IPN y UAM.\n\n"

        "📌 `&meme1`\n"
        "Envia el primer meme.\n\n"

        "📌 `&ayuda`\n"
        "Muestra todos los comandos.\n\n"

        "🌱 ¡Informate y ayuda a cuidar el planeta!"
    )

    await ctx.send(mensaje)


# ============================================
# 9. EVENTO CUANDO EL BOT SE CONECTA
# ============================================

@bot.event
async def on_ready():

    print("================================")
    print(f"BOT CONECTADO: {bot.user}")
    print("ESTADO: EN FUNCIONAMIENTO")
    print("================================")


# ============================================
# 10. MANEJO DE ERRORES
# ============================================

@bot.event
async def on_command_error(ctx, error):

    if isinstance(error, commands.CommandNotFound):

        await ctx.send(
            "❌ Comando no encontrado. "
            "Escribe `&ayuda`."
        )

    elif isinstance(error, commands.MissingRequiredArgument):

        await ctx.send(
            "❌ Faltan argumentos para ejecutar el comando."
        )

    else:

        print(f"ERROR: {error}")

        await ctx.send(
            "⚠️ Ocurrio un error al ejecutar el comando."
        )



TOKEN = "TU TOKEN"

bot.run(TOKEN)


 
