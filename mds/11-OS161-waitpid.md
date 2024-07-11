# Lab 4 — waitpid

- [Lab 4 — waitpid](#lab-4--waitpid)
  - [Overview](#overview)
  - [processTable](#processtable)
  - [proc\_search\_pid](#proc_search_pid)
  - [proc\_init\_waitpid](#proc_init_waitpid)
  - [proc\_end\_waitpid](#proc_end_waitpid)
  - [proc\_create](#proc_create)
  - [proc\_destroy](#proc_destroy)
  - [syscall.c](#syscallc)
  - [sys\_getpid](#sys_getpid)
  - [sys\_waitpid](#sys_waitpid)
  - [sys\_exit](#sys_exit)
  - [fork (EXTRA)](#fork-extra)


## Overview

![Untitled](../imgs/OS161waitpid-Untitled.png)

![Untitled](../imgs/OS161waitpid-Untitled%201.png)

![Untitled](../imgs/OS161waitpid-Untitled%202.png)

Su sistemi unix ci sono delle systemcall che permettono di attendere l’attesa della fine di processi figli o comunque di certi processi.
Come riconosciamo se il processo finito è quello che aspettiamo noi?
Tramite il ***pid***.

![Untitled](../imgs/OS161waitpid-Untitled%203.png)

![Untitled](../imgs/OS161waitpid-Untitled%204.png)

![Untitled](../imgs/OS161waitpid-Untitled%205.png)

![Untitled](../imgs/OS161waitpid-Untitled%206.png)

![Untitled](../imgs/OS161waitpid-Untitled%207.png)

---

L’obiettivo del laboratorio è implementare la syscall `waitpid` , che fa in modo che il kernel aspetti che un processo utente termini la sua esecuzione — o in generale che un processo aspetti il termine dell’esecuzione di un altro processo.

Dobbiamo capire che waitpid è una funzione signal/wait, che invece di aspettare la signal da parte di un altro processo/thread, aspetta la terminazione di un processo, dato un certo pid (process id).

Ciò significa che dobbiamo in qualche modo *generare* degli id per i nostri processi, e memorizzarli da qualche parte, per tenerne traccia.
Dobbiamo quindi implementare una ***process table***.

Attualmente, quando un processo termina la sua esecuzione, esso viene posto in stato di zombie, ma la sua struttura dati è ancora allocata, e non viene rilasciata.

![Untitled](../imgs/OS161waitpid-Untitled%208.png)

![Untitled](../imgs/OS161waitpid-Untitled%209.png)

![Untitled](../imgs/OS161waitpid-Untitled%2010.png)

![Untitled](../imgs/OS161waitpid-Untitled%2011.png)

Vediamo la soluzione del prof.

## processTable

![Untitled](../imgs/OS161waitpid-Untitled%2012.png)

Notiamo l’implementazione della suddetta process table.
All’interno troviamo:

- flag per capire se impostarla ad attiva o meno
- array di 100 puntatori a struttura di processo (in realtà 101, ma 0 non si usa)
- indice dell’ultimo pid allocato
- spinlock per la table

Notiamo inoltre che `processTable` è una variabile globale, quindi sempre disponibile.

## proc_search_pid

![Untitled](../imgs/OS161waitpid-Untitled%2013.png)

Restituisce un puntatore a `proc`, dato un pid.
In questa soluzione pid è l’index della processTable, quindi torniamo semplicemente `processTable.proc[pid]`.
Notiamo che lo facciamo *senza* mutual exclusion.

## proc_init_waitpid

![Untitled](../imgs/OS161waitpid-Untitled%2014.png)

La funzione riceve una struct proc, un nome e, in mutua esclusione (`spinlock_acquire(&processTable.lk`), fa una serie di cose:

- legge il primo indice “disponibile” nell’array dei processi (ultimo inserito+1)
- entra in un while che cerca per una posizione libera nell’array, e appena la trova vi assegna il processo, aggiorna `last_i` , e assegna l’indice `i`  come pid del processo.
Questo while fa “un giro” intero dell’array, fino a tornare all’indice `last_i`, nel caso in cui non trovasse prima una posizione libera nell’array

Dopo di che, se tutto è andato bene, dobbiamo gestire la sincronizzazione, dunque crea un semaforo (o una cv) sul processo.
Questo è un modo in cui possiamo fondamentalmente stabilire un collegamento tra un processo e un id numerico.

## proc_end_waitpid

![Untitled](../imgs/OS161waitpid-Untitled%2015.png)

La funzione semplicemente elimina dall’array il processo ricevuto come parametro, e distrugge il semaforo (o la cv).

## proc_create

![Untitled](../imgs/OS161waitpid-Untitled%2016.png)

Notiamo che nella funzione `proc_create`, già esistente, abbiamo ora inserito una chiamata a `proc_init_waitpid`.

## proc_destroy

![Untitled](../imgs/OS161waitpid-Untitled%2017.png)

![Untitled](../imgs/OS161waitpid-Untitled%2018.png)

Analogamente, nella `proc_destroy`, abbiamo inserito una chiamata a `proc_end_waitpid`.

## syscall.c

![Untitled](../imgs/OS161waitpid-Untitled%2019.png)

Notiamo che abbiamo modificato il file `syscall.c`, aggiungendo dunque il supporto per nuove syscalls, in particolare:

- **`SYS_waitpid`**
Prende in input un pid, un puntatore in user memory e un int
- **`SYS_getpid`**

Esaminiamo quindi tali syscalls.

## sys_getpid

![Untitled](../imgs/OS161waitpid-Untitled%2020.png)

La funzione restituisce il pid del *current process*.
Infatti ricordiamo che quando creiamo un processo, tramite `proc_create` , chiamiamo `proc_init_waitpid` , che al suo interno assegna un pid al processo creato.
Quindi è poi possibile leggerlo.

## sys_waitpid

![Untitled](../imgs/OS161waitpid-Untitled%2021.png)

Premessa: **non gestiamo le options**.
La funzione recupera il processo tramite il pid ricevuto come parametro (chiamando `proc_search_pid`) e chiama una funzione `proc_wait`.

Tale funzione semplicemente attende la terminazione di un processo, dato il puntatore a tale processo.

Dopo di che, recupera il codice di stato d’uscita del processo, ne distrugge la struttura (tramite `proc_destroy`) e ritorna il codice di stato.

Si noti come questa sia la prima volta che chiamiamo la funzione `proc_destroy`, mentre prima la struttura dati di un processo rimaneva allocata anche dopo la sua terminazione!

![Untitled](../imgs/OS161waitpid-Untitled%2022.png)

**Attenzione!**
Vediamo come, dopo aver atteso la terminazione del processo, recuperiamo il codice di stato tramite `proc->p_status` , il quale viene impostato dalla `sys_exit`!
Esaminiamo dunque la `sys_exit`.

## sys_exit

![Untitled](../imgs/OS161waitpid-Untitled%2023.png)

Vediamo che adesso è più complessa.
La funzione riceve lo status del processo.
Tale status viene mascherato con `0xff` , cioè ne prendiamo solo il byte meno significativo, e lo salviamo nella struttura dati del processo, nel campo `p->p_status`.
Dopo di ciò facciamo una signal per segnalare, a chi sta aspettando per la terminazione di questo processo, che lo stiamo terminando.
Dopo di che chiamiamo `thread_exit` che si occuperà di:

- “staccare” il thread dal processo 
(tramite `proc_remthread`, che semplicemente decrementa il counter dei threads appartenenti al processo)
- chiamare `thread_switch` 
che a sua volta si occupa di **cambiare lo stato** del thread corrente (mettendolo a zombie), **distruggere lo stack kernel level** del thread e **la struttura dati del thread** stesso, per poi **fare uno switch** ad un altro thread.

**Nota!**
Questa volta, al contrario di come facevamo prima di implementare la corretta terminazione di un processo, **non distruggiamo** l’address space del processo nella `sys__exit`.
Infatti, rimandiamo questa operazione nella `proc_wait`, che si occupa poi di distruggere il processo, insieme al suo address space richiamando `proc_destroy`, la quale appunto finora non si era utilizzata, e possiamo vedere che all’interno richiama `as_destroy`.

Possiamo ora capire meglio la funzione `proc_wait` 

![Untitled](../imgs/OS161waitpid-Untitled%2022.png)

Vediamo che quindi, una volta che è stata fatta la signal, usciremo dalla wait e proseguiremo nell’esecuzione, leggendo lo status del processo (precedentemente salvato nella struttura dati del processo dalla `sys_exit`) e distruggendo poi la struttura dati del processo.
Concludiamo restituendo lo status del processo terminato, al processo che ne attendeva la terminazione.

**Problema!**
Notiamo che nella nuova `sys__exit` , prima di fare la signal, chiamiamo `proc_remthread`, ma tale funzione è anche chiamata all’interno della `thread_exit` , che viene anch’essa chiamata dopo.
Questo è un problema, perchè ci troveremmo nel caso in cui `proc->p_numthreads` sarebbe minore di 0 (avendo chiamato proc_remthread 2 volte), facendo panicare il kernel.
Inoltre in realtà, alla seconda chiamata di `proc_remthread`, già la prima assert fallirebbe (`KASSERT(proc != NULL);`), in quanto la prima volta che si è chiamata avrebbe posto a NULL il riferimento al processo del thread.
Dobbiamo trovare un modo per *togliere* quella chiamata a `proc_remthread` nella `sys__exit` , oppure toglierla dalla `thread_exit` .
Ma innanzitutto: *perchè ci serve la chiamata a* `proc_remthread` *nella* `sys__exit`*?*

![Untitled](../imgs/OS161waitpid-Untitled%2024.png)

Supponiamo di levare la chiamata a `proc_remthread` nella `sys__exit`..
Dopo aver fatto la signal, quindi, chiameremo `thread_exit` , la quale internamente farà la sua chiamata a `proc_remthread`.
Ma **potrebbe** succedere che la `thread_exit` non sia abbastanza veloce da staccare il thread dal processo dopo aver fatto la signal. 
Cosa succederebbe quindi?

![Untitled](../imgs/OS161waitpid-Untitled%2022.png)

Succederebbe che il processo risvegliato, in attesa della terminazione del processo che aspettava, proseguirebbe nell’esecuzione, chiamando `proc_destroy` .

![Untitled](../imgs/OS161waitpid-Untitled%2025.png)

In `proc_destroy` ad un certo punto succede questo: un’assert su `p_numthreads == 0` .
Ma se la `thread_exit` non è abbastanza veloce a rimuovere il thread dal processo *prima* che ciò accada, la `proc_destroy` tenterebbe comunque a distruggere il processo (che ha ancora un thread attaccato), l’assert fallirebbe e il kernel panicherebbe.

Per questo motivo si era inserita quella chiamata a `proc_remthread` *prima* della `thread_exit`.

**Soluzione!**

![Untitled](../imgs/OS161waitpid-Untitled%2026.png)

Modifichiamo la `thread_exit`: se quando chiamiamo la `thread_exit`, il thread corrente ha ancora un riferimento al processo, allora chiamiamo `proc_remthread` e li disaccoppiamo.
Altrimenti procediamo assumendo che appunto il processo non ha più threads attaccati.

![Untitled](../imgs/OS161waitpid-Untitled%2027.png)

Inoltre modifichiamo la funzione `common_prog`, la quale si occupa di avviare un processo utente, aspettandone ora l’esecuzione e terminazione, leggendone il codice di stato e ritornandolo.
Possiamo ora vedere che torniamo al menu di OS161 solo **dopo** aver finito l’esecuzione di un programma utente, ad esempio `testbin/palin` 

![Untitled](../imgs/OS161waitpid-Untitled%2028.png)

## fork (EXTRA)

![Untitled](../imgs/OS161waitpid-Untitled%2029.png)

Vediamo la soluzione proposta per la syscall `sys_fork` del prof.
Questa syscall consente di generare un processo a partire da un altro processo.
Essenzialmente crea un programma utente tramite `proc_create_runprogram` , la quale a sua volta crea la struttura dati del processo tramite `proc_create` e la inizializza.
Poi, copia l’address space del processo padre, e lo assegna al nuovo processo.
Copia anche la trapframe del processo padre, e la assegna al nuovo processo.
Poi esegue la `thread_fork`, però al contrario di quanto accade in `common_prog` quando eseguiamo un nuovo programma, in cui, tramite `cmd_progthread` e poi `runprogram`, carichiamo un file eseguibile dato che creiamo un processo nuovo da zero, qui abbiamo clonato un altro processo e quindi la funzione che il nuovo thread esegue è `call_enter_forked_process`.

![Untitled](../imgs/OS161waitpid-Untitled%2030.png)

La funzione semplicemente fa un cast a (trapframe *) (che era stato passato come (void *) nella `thread_fork`) e chiama `enter_forked_process` passandole il trapframe.

![Untitled](../imgs/OS161waitpid-Untitled%2031.png)

Tale funzione copia il trapframe nel kernel stack del nuovo thread, e poi avvia effettivamente il thread.

Si riportano le funzioni chiamate da `common_prog` per comparazione.

![Untitled](../imgs/OS161waitpid-Untitled%2032.png)

![Untitled](../imgs/OS161waitpid-Untitled%2033.png)

![Untitled](../imgs/OS161waitpid-Untitled%2034.png)