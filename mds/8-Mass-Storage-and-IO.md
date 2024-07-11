# Mass Storage and I/O

- [Mass Storage and I/O](#mass-storage-and-io)
- [I/O Systems](#io-systems)


> ***Premessa***
*La lezione **Mass Storage and I/O** è abbastanza semplice e di carattere generale, inoltre non stavo bene quando l’ho seguita, dunque non ho preso particolarmente appunti dato che le slides sono molto semplici, autoesplicative e sufficienti per capire i concetti spiegati.*
> 

![Untitled](../imgs/MassStorage1-Untitled.png)

![Untitled](../imgs/MassStorage1-Untitled%201.png)

![Untitled](../imgs/MassStorage1-Untitled%202.png)

![Untitled](../imgs/MassStorage1-Untitled%203.png)

Tempo necessario per un’operazione di I/O dipende non solo dall’Avarage access time, ma anche dal tempo che impieghiamo a spostare i dati dal disco alla CPU.
Vediamo quindi che sono coinvolti anche altri due tempi: 

- **`amount to transfer / transfer rate`**  → Rapporto tra *quanti dati devo trasferire e velocità di trasferimento*
- **`controller overhead`** → I dispositivi di I/O non possono parlare direttamente con la CPU. Il controller è un un circuito integrato che permette al dispositivo di parlare con il SO. Il *controller overhead* è il tempo che impiega il controller a gestire le operazioni.

Inoltre, poichè il settore che dobbiamo leggere può essere “appena passato”, e quindi impiegare un giro completo prima di essere letto, oppure può essere che il primo settore che sta passando sia proprio quello che dobbiamo leggere, poniamo come ***Avarage latency = latency / 2.***

![Untitled](../imgs/MassStorage1-Untitled%204.png)

![Untitled](../imgs/MassStorage1-Untitled%205.png)

![Untitled](../imgs/MassStorage1-Untitled%206.png)

![Untitled](../imgs/MassStorage1-Untitled%207.png)

![Untitled](../imgs/MassStorage1-Untitled%208.png)

![Untitled](../imgs/MassStorage1-Untitled%209.png)

![Untitled](../imgs/MassStorage1-Untitled%2010.png)

![Untitled](../imgs/MassStorage1-Untitled%2011.png)

![Untitled](../imgs/MassStorage1-Untitled%2012.png)

![Untitled](../imgs/MassStorage1-Untitled%2013.png)

![Untitled](../imgs/MassStorage1-Untitled%2014.png)

![Untitled](../imgs/MassStorage1-Untitled%2015.png)

![Untitled](../imgs/MassStorage1-Untitled%2016.png)

![Untitled](../imgs/MassStorage1-Untitled%2017.png)

![Untitled](../imgs/MassStorage1-Untitled%2018.png)

![Untitled](../imgs/MassStorage1-Untitled%2019.png)

![Untitled](../imgs/MassStorage1-Untitled%2020.png)

![Untitled](../imgs/MassStorage1-Untitled%2021.png)

![Untitled](../imgs/MassStorage1-Untitled%2022.png)

![Untitled](../imgs/MassStorage1-Untitled%2023.png)

![Untitled](../imgs/MassStorage1-Untitled%2024.png)

![Untitled](../imgs/MassStorage1-Untitled%2025.png)

![Untitled](../imgs/MassStorage1-Untitled%2026.png)

![Untitled](../imgs/MassStorage1-Untitled%2027.png)

![Untitled](../imgs/MassStorage1-Untitled%2028.png)

![Untitled](../imgs/MassStorage1-Untitled%2029.png)

![Untitled](../imgs/MassStorage1-Untitled%2030.png)

![Untitled](../imgs/MassStorage1-Untitled%2031.png)

![Untitled](../imgs/MassStorage1-Untitled%2032.png)

![Untitled](../imgs/MassStorage1-Untitled%2033.png)

# I/O Systems

![Untitled](../imgs/MassStorage1-Untitled%2034.png)

![Untitled](../imgs/MassStorage1-Untitled%2035.png)

![Untitled](../imgs/MassStorage1-Untitled%2036.png)

![Untitled](../imgs/MassStorage1-Untitled%2037.png)

![Untitled](../imgs/MassStorage1-Untitled%2038.png)

![Untitled](../imgs/MassStorage1-Untitled%2039.png)

![Untitled](../imgs/MassStorage1-Untitled%2040.png)

![Untitled](../imgs/MassStorage1-Untitled%2041.png)

![Untitled](../imgs/MassStorage1-Untitled%2042.png)

![Untitled](../imgs/MassStorage1-Untitled%2043.png)

![Untitled](../imgs/MassStorage1-Untitled%2044.png)

![Untitled](../imgs/MassStorage1-Untitled%2045.png)

![Untitled](../imgs/MassStorage1-Untitled%2046.png)

![Untitled](../imgs/MassStorage1-Untitled%2047.png)

![Untitled](../imgs/MassStorage1-Untitled%2048.png)

![Untitled](../imgs/MassStorage1-Untitled%2049.png)

![Untitled](../imgs/MassStorage1-Untitled%2050.png)

![Untitled](../imgs/MassStorage1-Untitled%2051.png)

![Untitled](../imgs/MassStorage1-Untitled%2052.png)

![Untitled](../imgs/MassStorage1-Untitled%2053.png)

![Untitled](../imgs/MassStorage1-Untitled%2054.png)

![Untitled](../imgs/MassStorage1-Untitled%2055.png)

![Untitled](../imgs/MassStorage1-Untitled%2056.png)

![Untitled](../imgs/MassStorage1-Untitled%2057.png)

![Untitled](../imgs/MassStorage1-Untitled%2058.png)

![Untitled](../imgs/MassStorage1-Untitled%2059.png)
