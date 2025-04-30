from microdot import Microdot, send_file, Response
import machine
import onewire, ds18x20
import time

def connect_to(ssid: str, passwd: str) -> None:
    import network
    from time import sleep

    sta_if = network.WLAN(network.STA_IF)
    if not sta_if.isconnected():
        print("Connecting to network...")
        sta_if.active(True)
        sta_if.connect(ssid, passwd)
        while not sta_if.isconnected():
            print(".", end="")
            sleep(0.05)

    print("\nNetwork config:", sta_if.ifconfig(), "\n")

# Conexión WiFi
connect_to("Cooperadora Alumnos", "")

# Servidor web
app = Microdot()
Response.default_content_type = 'application/json'

# LEDs
led1 = machine.Pin(32, machine.Pin.OUT)
led2 = machine.Pin(33, machine.Pin.OUT)
led3 = machine.Pin(25, machine.Pin.OUT)
led4 = machine.Pin(14, machine.Pin.OUT)

# Sensor de luz (LDR)
ldr = machine.ADC(machine.Pin(39))
ldr.atten(machine.ADC.ATTN_11DB)
ldr.width(machine.ADC.WIDTH_10BIT)

# Botones físicos
izq = machine.Pin(13, machine.Pin.IN)
ent = machine.Pin(15, machine.Pin.IN)
der = machine.Pin(23, machine.Pin.IN)

# Sensor de temperatura DS18B20
ds_pin = machine.Pin(19)
ds_sensor = ds18x20.DS18X20(onewire.OneWire(ds_pin))
roms = ds_sensor.scan()
print("Sensores DS18B20 encontrados:", roms)

# === RUTAS MICRODOT ===

@app.route("/")
async def index(request):
    return send_file("index.html", content_type="text/html")

@app.route("/ldr")
def leer_luz(request):
    valor = ldr.read()
    porcentaje = int((valor / 1023) * 100)
    return {"ldr": porcentaje}

@app.route("/temp")
def leer_temp(request):
    ds_sensor.convert_temp()
    time.sleep_ms(750)  # Tiempo de conversión típico para DS18B20
    if roms:
        temp_c = ds_sensor.read_temp(roms[0])
        return {"temp": round(temp_c, 2)}
    else:
        return {"error": "No DS18B20 sensor found"}, 500

@app.route("/led/on/<led>")
async def led_on(request, led):
    if led == "1":
        led1.on()
    elif led == "2":
        led2.on()
    elif led == "3":
        led3.on()
    elif led == "4":
        led4.on()
    return {"status": "ok", "led": led, "action": "on"}

@app.route("/led/off/<led>")
async def led_off(request, led):
    if led == "1":
        led1.off()
    elif led == "2":
        led2.off()
    elif led == "3":
        led3.off()
    elif led == "4":
        led4.off()
    return {"status": "ok", "led": led, "action": "off"}

@app.route("/switches")
async def switches(request):
    return {
        "izq": izq.value(),
        "ent": ent.value(),
        "der": der.value()
    }

# Inicia el servidor  
app.run()
