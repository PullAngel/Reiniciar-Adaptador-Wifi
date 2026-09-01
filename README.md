# Reiniciar Adaptador Wi-Fi

Un `.cmd` liviano para reiniciar el adaptador Wi-Fi en Windows 10 y 11 sin tener que reiniciar toda la PC.

Busca adaptadores Wi-Fi físicos, los deshabilita desde Windows, espera 5 segundos y los vuelve a habilitar. Después Windows intenta reconectar la red por su cuenta. Contiene Arte ASCII.

```text
          .       *                  .--------------------------------------------------.
               _/\_                  |              INSTRUCCIONES DE USO               |
         *   .'    `.          .----< |  1. Abri el archivo como administrador.         |
            /  /\ /\  \              |  2. Espera unos segundos.                        |
      _____/__/  V  \__\_____        |  3. Listo.                                        |
     /_____.-' .---. `-.____\        '--------------------------------------------------'
           |  / o o \  |
           |  \  ^  /  |
            \  `===`  /
         _.-'`-.___.-'`-._
       .'   _    /\    _  `-.
      /   .' \  /  \  / `.   \
     /   /    \/ /\ \/    \   \
    /___/      _/  \_      \___\
      /       /      \       \
     /   _   /   /\   \   _   \
    /___/ \_/___/  \___\_/ \___\
         _/  /      \  \_
        /___/        \___\
```

## Uso rapido

1. Descarga [Reiniciar-WiFi.cmd](./Reiniciar-WiFi.cmd).
2. Ejecuta en modo administrador.
3. Espera a que termine. Se cerrará solo.

El archivo detecta el adaptador automáticamente. No contiene el identificador fijo de ningún dispositivo, así que se puede usar en cualquier PC con Windows.

## Comando limpio para PowerShell

Recomiendo la experiencia del `.cmd`, que además contiene caminos alternativos a errores con algunos
adaptadores wifi. Sobre todo si necesitas reiniciarlo de forma constante, es más cómodo tenerlo a mano.
Pero si quieres hacerlo de forma manual por vía terminal puedes ejecutar en powershell con permisos de administrador el siguiente bloque:

```powershell
$ErrorActionPreference = 'Stop'
$ProgressPreference = 'SilentlyContinue'

$adapters = @(Get-NetAdapter -Physical -IncludeHidden | Where-Object {
    $_.NdisPhysicalMedium -eq '802.11' -and $_.PnPDeviceID
})

if (-not $adapters) {
    $adapters = @(Get-NetAdapter -Physical -IncludeHidden | Where-Object {
        ($_.Name -match '(?i)wi-?fi|wireless|wlan|802\.11' -or
         $_.InterfaceDescription -match '(?i)wi-?fi|wireless|wlan|802\.11') -and
        $_.PnPDeviceID
    })
}

if (-not $adapters) {
    throw 'No se encontro un adaptador Wi-Fi fisico.'
}

foreach ($adapter in $adapters) {
    Disable-PnpDevice -InstanceId $adapter.PnPDeviceID -Confirm:$false -ErrorAction Stop | Out-Null
    Start-Sleep -Seconds 5
    Enable-PnpDevice -InstanceId $adapter.PnPDeviceID -Confirm:$false -ErrorAction Stop | Out-Null
}
```

Este comando reinicia el dispositivo Wi-Fi y su controlador. No garantiza que Internet vuelva de inmediato: Windows todavía necesita reconectarse a la red.

## Requisitos

- Windows 10 o Windows 11.
- PowerShell 5.1 o posterior.
- Permisos de administrador.
