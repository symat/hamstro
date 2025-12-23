# temporary notes

## MCU parameters
- AVR64DU28
- 8 KB RAM
- 64 KB flash

## program load and execution

Three modes of operation:
- 



AVR64DU
8 KB RAM
64 KB flash
+ SD

flash: operációs rendszer = loader + common lib-ek, összesen 64 KB

SD: programok, köztük az os (common_libs és a loader is), hogy vissza lehessen állítani

loader: kb 8 KB, a flash elején
- ha USB-n csatlakoztatva van, akkor mint külső eszköz csatolódik és várja a fájlátvitelt
  - ha a fájlt boot.bin-nek hívják, akkor az internal flash-re megy és az első 8 KB
  - ha lib_VVVV.bin-nek hívják, akkor az internal flash-re megy, a felső 56 KB-ra,
    a VVVV valamilyen library verzió/azonosító
  - ha os.bin-nek hívják, akkor az internal flash-re megy és full 64 KB-os lehet 
    (az eleje ilyenkor a loader, a vége a common library)
  - ha akár boot, libs vagy os, akkor kimentődik az SD kártyára is a megfelelő helyekre
  - ha egyik sem, akkor az SD-re menne az applications alá
  - kiterjesztések:## 
     - bin kiterjesztés: bináris program, full-load
	 - bim kiterjesztés: bináris program, mem-load
	 - bil kiterjesztés: bináris program, lib-load (a fájlnév utolsó 4 karaktere a lib azonosító)
	 - txt kiterjesztés: leírás
	 - bmp kiterjesztés: screenshot
  - ha van SD, akár mutathatja is az SD kártyát is USB-n keresztül? ez nem tudom, belefér-e 8KB-ba
- ha van SD kártya, akkor azt listázza a képernyőn a programokat
- ha program van betöltve (full-load), akkor indíthatjuk a betöltött programot
- vagy választhatunk valamilyen beállítós / diag progikat is, ha beleférnek a loader-be

az eeprom-ban, vagy a flash elején valahol (?) van egy config rész:
- mi van most betöltve (full-app, vagy common_libs?)
- esetleg milyen verziók? (loader verzió, common_lib verzió)


az SD kártyán vannak a programok. Amikor programot tültünk be, akkor lehet:
- mem-load:  kicsi programoknak, amik beleférnek 8KB-ba, ezek a memóriába töltődnek
             nem terhelik az internal flash-t és gyorsabbak is. 
- lib-load:  kicsi programoknak, amik beleférnek 8KB-ba, ezek a memóriába töltődnek
             de használják a flash-en lévő common lib-eket. Ezért ha ezek nincsenek 
			 betöltve, akkor először be kell őket tölteni és csak utána indul a memóriából
			 a program.
- full-load: ez a teljes flash-t (kivéve az első 8 KB) felülcsapja, bonyibb programokra van. 
             felülírja a common lib-eket


minden progi 
