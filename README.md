# 🖥️ HomeLab

Totul a început cu un **Asus VivoBook** care aduna praf de ceva timp. Deși eare un procesor capabil, era considerat "depășit" pentru workflow-ul zilnic modern. Inspirația a venit explorând comunitățile de _HomeLab_, unde am realizat că hardware-ul "vechi" este, de fapt, o resursă neexploatată care poate găzdui un întreg ecosistem de servicii și automatizări.

Am decis să îi ofer o a doua șansă și o misiune critică: transformarea dintr-un laptop care stătea degeaba în casă într-un **Server de Virtualizare Proxmox**.

----
## 🛠️ Provocarea Hardware:

Cea mai mare provocare a acestui proiect a fost optimizarea resurselor. Într-o eră a serverelor cu sute de GB de RAM, acest proiect demonstrează eficiență maximă pe o configurație modestă, dar echilibrată:

### 🛠️ Hardware:

Nu am avut nevoie de un rack întreg de servere. Totul rulează pe un laptop Asus cu următoarele specificații, optimizate la sânge:

| Componentă  | Specificație                                  | Rol în ecosistem                                                                                                                                                 |
| :---------- | :-------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CPU**     | **Intel Core i7-10510U** (4 Cores, 8 Threads) | Se ocupă de calculele pentru LLM și n8n, dar și de transcodarea video în Jellyfin.                                                                               |
| **RAM**     | **16 GB DDR3**                                | Este împărțită strategic între VM-ul de AI (care „înghite” cel mai mult) și containerele LXC.                                                                    |
| **Storage** | **SSD** (Boot + VM/Containers Storage)        | SSD-ul se ocupă de încărcarea rapidă a modelelor de AI (care sunt masive) și asigură că n8n procesează automatizările instant, fără să stea după scriere/citire. |

---

## 🎮 Serviciile:

Fiecare serviciu are rolul lui bine stabilit. Am ales să combin **Mașini Virtuale (VM)** pentru izolare maximă și **Containere (LXC)** pentru viteză și consum redus de resurse.

<img width="2870" height="1044" alt="image" src="https://github.com/user-attachments/assets/2ddab235-2501-4eff-b2b4-96a3b7095017" />


---
### 🧠 1. Local LLM (VM 117)

Aici se întâmplă partea mea favorită. Este o mașină virtuală dedicată pentru un **LLM local**, rulat prin `llama-cpp`

<img width="2874" height="1036" alt="image" src="https://github.com/user-attachments/assets/2a039612-b86d-4cbf-8b6c-15bc39353159" />

- **De ce VM?** Pentru că LLM-ul are nevoie de resurse „rezervate” și de o izolare mai bună la nivel de kernel.
- **Setup-ul:** Am alocat **10GB de RAM**  pentru a putea încărca modele de limbaj decente.
- **Performanță:** Folosesc versiuni GGUF ale modelelor pentru a scoate maximum de viteză din procesorul i7, fără să am nevoie de o placă video dedicată.

<img width="2880" height="1552" alt="image" src="https://github.com/user-attachments/assets/4e918833-c6ed-4e18-83b5-ec4e6b29d6d3" />

Folosesc un model **Qwen3 de 4 miliarde de parametri**, special antrenat pentru "Reasoning" (gândire logică complexă), care a fost distilat pentru a oferi performanțe similare modelelor mult mai mari. Este în format **GGUF cu cuantizare Q4_K_M**, astfel modelul are o dimensiune redusă de doar **2.32 GB**, ceea ce îi permite să ruleze extrem de rapid pe procesorul i7, lăsând destulă memorie RAM liberă și pentru celelalte servicii de pe server.

![llm](https://github.com/user-attachments/assets/5e444ce4-d9c1-406a-bbf3-035a79a4a0cb)

### ⚡ Viteza de răspuns:

Să rulezi un model de AI pe un procesor de laptop (fără placă video dedicată) poate suna lent, dar optimizarea prin **GGUF** și cuantizarea **Q4_K_M** fac minuni. Pe setup-ul meu, vitezele de generare sunt surprinzător de utilizabile:

- **7-8 tokens/secundă (Task-uri ușoare):** Această viteză este comparabilă cu ritmul în care citește un om obișnuit. Textul apare pe ecran natural, ca și cum cineva ar tasta rapid în timp real. Pentru întrebări simple, sumarizări sau formatat date, nu simți nicio întârziere deranjantă.

- **3-4 tokens/secundă (Task-uri complexe/Reasoning):** Modelul meu, **Qwen3-4B-Thinking**, este unul de tip "reasoning". Chiar dacă viteza scade la jumătate, calitatea răspunsului este mult mai mare. Pentru rezolvat probleme de logică sau debug de cod, prefer să aștept 2 minute în plus pentru un răspuns corect, decât să primesc unul rapid și greșit.
---

### 🗺️ 2. n8n workflow (CT 100)

L-am instalat într-un container LXC pentru că este incredibil de eficient: pornește instant și ocupă foarte puține resurse.


<img width="2880" height="960" alt="image" src="https://github.com/user-attachments/assets/68ce9697-a7c8-4f70-baa3-37b9b5ec6457" />
### ⚙️ Specificații Container (LXC)

- **Resurse:** 2 vCPU | 2 GB RAM - poate părea puțin, dar chiar sunt arhisuficiente (probabil s-ar descurca la fel de bine si cu 1 vCPU si 1 GB RAM).
- **Network:** Integrat în bridge-ul Proxmox pentru a comunica rapid cu restul serviciilor locale.

### ❓ De ce n8n și nu scripturi Python?

- **Vizual:** Pot vedea exact pe unde trec datele și unde se blochează ceva (dacă e cazul).
- **Integrare:** Are sute de noduri gata făcute pentru Telegram, Discord, Google Sheets sau cereri HTTP către propriul meu VM de AI.
- **Viteză:** Modific un workflow instant, fără să dau restart la vreun serviciu.

---

### 🎬 3. Jellyfin (CT 111)

Este alternativa mea open-source la Netflix. Am control total asupra librăriei mele media, zero reclame și o interfață super curată care își ia singură toate detaliile despre filme (postere, descrieri, trailere).


<img width="2874" height="924" alt="image" src="https://github.com/user-attachments/assets/37d38051-ea58-4899-a8c9-c794618d4b85" />
#### ⚙️ Specificații Container (LXC)

- **Resurse:** 2 vCPU | 2 GB RAM — Deși pare puțin, containerul este extrem de eficient pentru streaming.

#### ❓ De ce Jellyfin și cum l-am testat?

- **Testul de performanță:** Momentan nu am adăugat o librărie de filme/seriale. Pentru a verifica dacă totul este configurat corect, am încărcat 1-2 filmări personale de test.

- **Rezultatul:** Streaming-ul a rulat impecabil, fără niciun pic de lag sau buffering, confirmând că alocarea de resurse și accesul la disk sunt optime.
---
### 🌐 4. Tailscale

Tailscale creează o rețea VPN mesh între toate dispozitivele mele, fără să ma ating de router sau să deschid vreun port. Fiecare dispozitiv primește o adresă IP fixă din rețeaua privată Tailscale, iar comunicarea dintre ele este criptată end-to-end.
#### De ce este cea mai eficientă soluție:

- **Zero configurație pe router:** Nu m-am atins de setările de rețea ale router-ului. Tailscale trece prin orice firewall și creează un tunel securizat între dispozitivele mele.

- **Remote Control:** Dacă am nevoie de LLM-ul local sau orice alt serviciu de pe server, îl pot accesa direct de pe telefon.

<img width="1170" height="1164" alt="image" src="https://github.com/user-attachments/assets/beb87fff-0ddf-4357-a63e-91bb9d8dba9b" />

---
