# TDOpenStage

## Estado del componente

Este es un componente en desarrollo. Por ahora lo serviré desde Google Drive y tendrá integrado en el .zip una 
versión de Open Stage Control para facilitar el uso inmediato, pero sobre todo, para ilustrar cómo incluir dentro 
de un proyecto de TouchDesigner una dependencia externa.

Insisto, esto será temporal hasta que aparezca definitivamente en este repositorio. 
[Aquí](https://drive.google.com/file/d/12WXIoLE-C2LqLeh4CimnBQ-0jDjMfmyp/view?usp=sharing) está el proyect/componente.


## Configuración del servidor open stage control

En este componente se lanza el servidor de open stage control. En la documentación oficial se pueden consultar los parámetros que requiere el servidor al momento 
de lanzarlo desde la terminal: [Configuración del servidor](https://openstagecontrol.ammd.net/docs/getting-started/server-configuration/)

<img width="710" height="497" alt="image" src="https://github.com/user-attachments/assets/f2234dda-6899-4855-87ce-05be21c5f020" />

El puerto 9000 corresponde al **send** que tu configuras en launcher de Open Stage Control cuando haces pruebas. En este puerto debe estar 
escuchando TouchDesigner, u otro software. En puerto 9001 corresponde al parámetro **osc-port**. Este será el puerto al que TouchDesigner 
deberá enviar mensajes OSC de regreso.

Cuando presiones Launch OpenStage Control se activará un **Paramenter Execute** en **OpenStageControlInOut**

<img width="998" height="471" alt="image" src="https://github.com/user-attachments/assets/5953b694-6931-4b47-b2f3-142a1cd9a055" />

Esta es la función asociada al evento de Pulse:

``` py
def onPulse(par: Par):
	"""
	Called when a parameter is pulsed.
	
	Args:
		par: The Par object that was pulsed
	"""
	project_path = project.folder
	# print(f"Parameter {par.name} was pulsed. Project path: {project_path}")
	raw_path = os.path.join(project_path, 'OpenStageControl', 'open-stage-control.exe')
	print(f"Raw path: {raw_path}")
	osc_exe = os.path.normpath(raw_path)
	print(f"Normalized path: {osc_exe}")
	session_file = os.path.normpath(os.path.join(project_path, 'OpenStageControl/myProjects/ambrosiaExplore1/clientUI.json'))
	print(f"Session file: {session_file}")
	oscPort = parent().par.Tdinport.eval()
	print(f"OSC Port: {oscPort}")
	oscAddress = f'127.0.0.1:{oscPort}'
	oscReceivePort = str(parent().par.Openstageinport.eval())
	print(f"OSC Receive Port: {oscReceivePort}")
	args = [
        osc_exe, 
        '--load', session_file, 
        '--send', oscAddress,
        '--port', '8086',
		'--osc-port', oscReceivePort
    ]

	try:
		subprocess.Popen(args)
		print("Open Stage Control launched successfully.")
	except Exception as e:
		print(f"Failed to launch Open Stage Control: {e}")
	return
```

## Bind de parámetros

En el Base Visuals hay un Bind que está configurado internamente 

<img width="641" height="418" alt="image" src="https://github.com/user-attachments/assets/45c554fd-fb8c-4917-8c6c-7823dcee82e3" />

<img width="992" height="397" alt="image" src="https://github.com/user-attachments/assets/c1d257d2-e976-4ba0-bda7-25f9c6acff5e" />

¿Qué es y para que sirve ese Bind?

Revisar esta [documentación oficial](https://docs.derivative.ca/Bind_CHOP).

## ¿Cómo hacer el bind hacia open stage control?

Aquí el reto es que al controlar un parámetro en TD también se actualice la interfaz de usuario del cliente de Open Stage Control.

La clave está en esta parte de la red:

<img width="966" height="298" alt="image" src="https://github.com/user-attachments/assets/0cbcc8d1-47d6-479d-84c5-10dff1dbfd49" />

Cada que cambia un parámetro en el Base Visuals **parexec1** se ejecuta y lo informa usando como un out DAT. Esta tabla ingresa a 
**OpenStageControlInOut**:

<img width="934" height="562" alt="image" src="https://github.com/user-attachments/assets/866bbaf2-b563-421d-a814-c9888c29e565" />

**datexec1** determinará si es necesario o no reenviar a Open Stage Control. Esto solo lo hará si los datos cambian desde TouchDesigner.  
Si los datos vienen de Open Stage Control, los parámetros de Visual cambiarán, pero **datexec1** no lo reportará.




