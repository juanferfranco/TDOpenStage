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

<img width="665" height="349" alt="image" src="https://github.com/user-attachments/assets/9ecff103-bf52-4e25-b56a-1eba2f13c277" />

En ShowControlIn hay un DAT asociado al parámetro Launchopenstage:

<img width="857" height="436" alt="image" src="https://github.com/user-attachments/assets/cb905534-4bc0-4188-a094-658ebac4c5f5" />

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

Nótese en la imagen que hay un Bind de los parámetros que llegan del cliente de Open Stage Control y la UI del Base Visuals del propio Touch.

<img width="1677" height="703" alt="image" src="https://github.com/user-attachments/assets/6410235b-2b34-4ec4-9d58-2013364be331" />

¿Qué es y para que sirve ese Bind?

Revisar esta [documentación oficial](https://docs.derivative.ca/Bind_CHOP).

## ¿Cómo hacer el bind hacia open stage control?

Aquí el reto es que al controlar un parámetro en TD también se actualice la interfaz de usuario del cliente de Open Stage Control.



