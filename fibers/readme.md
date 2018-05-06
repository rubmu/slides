# Quantum Mechanics of Calculations

Dmitriy Karlovskiy @ HolyJS 2018 Piter

# Issue: Low responsiveness

![Низкая отзывчивость](low-fps.gif)

# Issue: No escape

![Нельзя отменить](no-escape.jpg)

# Issue: No concurrency

![Быстрые ждут медленных](no-concurrency.gif)

# Solution: React Fiber

![Хитрая логика React Fiber](react-logic.png)

# React Fiber: Issues

- React required
- Only rendering
- Quantization is disabled by default
- Debug is pain

# Solution: Workers

![Логика работы с Workers](worker-logic.png)

# Workers: Issues

- (De)Serialization 
- Asynchronous
- Limited API’s
- Can’t cancel

# Solution: quantization

![Долгая задача без квантизации](flame-sync.png)
![Долгая задача с квантизацией](flame-quant.png)

# Concurrency: FSM – stackless coroutines

Properties:
- Async/await virus

Examples:
- C#
- Python
- JavaScript

# Concurrency: fibers – stackfull coroutines

Properties:
- Runtime support required

Examples:
- node-fibers
- Python
- Go
- D

# Concurrency: semi-fibers - restarts

Properties:
- Idempotent calculations
- Runtime support is not required

Examples:
- Future-fetcher
- $mol_atom
- $mol_fiber

# Figures

![Типичная диаграмма исполнения](sync-0.svg)

# $mol_fiber: no quantization

![Исполнение без квантизации](sync-1.svg)

# $mol_fiber: cache first

![Заполнение кешей](cache-1.svg)

# $mol_fiber: cache second

![Реиспользование кешей](cache-2.svg)

# $mol_fiber: idempotence first

![заполнение идемпотентных кешей](idemp-1.svg)

# $mol_fiber: idempotence second

![Реиспользование идемпотентных кешей](idemp-2.svg)

# Debug: try/catch

```typescript
function foo() {
	throw new Error( 'Something wrong' ) // 1?
}

try {
	foo()
} catch( error ) {
	handle( error )
	throw error // 2?
} 
```

# Debug: unhandled events

```typescript
function foo() {
	throw new Error( 'Something wrong' )
}

window.addEventListener( 'error' , event => handle( event.error ) )
window.addEventListener( 'unhandledRejection' , event => handle( event.reason ) )

process.on( 'uncaughtException' , error => handle( error ) )

foo()
```

# Debug: Promise

```typescript
function foo() {
	throw new Error( 'Something wrong' )
}

new Promise( ()=> {
	foo()
} ).catch( error => handle( error ) ) 
```

# Stack trace: React Fiber

![Бессодержательный стектрейс](react-debug.png)

# Stack trace: $mol_fiber

![Содержательный стектрейс](fiber-debug.png)


# $mol_fiber: functions

```typescript
import { $mol_fiber_func as fiberize } from 'mol_fiber/web'

const log = fiberize( console.log )

export const main = fiber( ()=> {
	log( getData( 'goo.gl' ).data )
} ) 
```

# $mol_fiber: promises

```typescript
import { $mol_fiber_sync as sync } from 'mol_fiber/web'

export const getData = sync( fetch )

export const getData = sync( async( ... args )=> {
	return ( await fetch( url ) ).data
} ) 
```

# $mol_fiber: methods

```typescript
import { $mol_fiber_method as action } from 'mol_fiber/web'

export class Mover {

	@action
	move() {
		sendData( 'ya.ru' , getData( 'goo.gl' ) )
	}

} 
```

# $mol_fiber: cancel request

```typescript
import { $mol_fiber_async as async } from 'mol_fiber/web'

function getData( uri : string ) : XMLHttpRequest { return async( back => {

	const xhr = new XMLHttpRequest()
	xhr.open( 'GET', uri )

	xhr.onload = back( event => {
		if( Math.floor( xhr.status / 100 ) === 2 ) return xhr
		throw new Error( xhr.statusText )
	} )
	xhr.onerror = back( event => { throw new Error( xhr.statusText ) } )

	xhr.send()
	
	return ()=> xhr.abort()
} ) }
```

# $mol_fiber: cancel response

```typescript
import { $mol_fiber_func as fiberize , $mol_fiber_make as Fiber } from 'mol_fiber'

const hash = fiberize( longHashFunc )

const middle_fiber = middleware => ( req , res ) => {
	const fiber = Fiber( ()=> middleware( req , res ) )
	req.on( 'close' , ()=> fiber.destructor() )
	fiber.start()
}

app.get( '/hash/:data/:count' , middle_fiber( ( req , res ) => {
	let data = req.params.data
	for( let i = 0 ; i < req.params.count ; ++ i ) data = hash( data )
	res.end( data )
} ) )
```

# $mol_fiber: concurrency

![Нет конкуренции](server-sync.png)
![Есть конкуренция](server-quant.png)

# $mol_fiber: properties

Pros:
- Runtime support isn’t required
- Can be cancelled at any time
- High FPS
- Concurrent execution
- Usefull stack trace
- ~ 10kb not minified

Cons:
- Required instrumentation
- All code should be idempotent
- Longer total execution