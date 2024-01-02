Readme
## Fusion_Lider_und_Kamera (Early fusion):


#### Schritt 1: Installation der Abhängigkeiten:
<ul>
<li>Google Drive wird mit dem Google Colab verbunden.</li>

<li>Die Datendateien (Points, Bilder und Kalibrierungsdateien) werden mit dem folgenden Code heruntergeladen.</li>
</ul>

```bash
!gdown 15weNDHn9lNUM1-UcMCBDyAd4RLFhCfpk
!unzip -q data.zip
```
```bash
!gdown 1tzQuDBm31ICPi4MXxTRVstQ0_1vQnugh
!unzip -q Yolov4.zip
```
#### Installation:
```bash
!pip install open3d # Version 12
```

```bash
!pip install pypotree #https://github.com/centreborelli/pypotree
```

#### Schritt 2: Die Points im Bild projizieren:

<p>Die 3D-Punkte sollen in das Bild projiziert werden.</p>

<p>Projektion Prozess:</p>

<p float="left">
  <img src="https://gist.github.com/assets/125800618/49eebb32-bb2d-4da0-a083-cf0d7142fd8a.png" width="1190" height="460" />

#### 2.1: Die Kalibrierungsdatei lesen:

<p>Der Schritt besteht darin, die Kalibrierungsdateien zu lesen. Für jedes Bild haben wir eine zugehörige Kalibrierungsdatei, die folgende Informationen enthält:</p>

<ul>
<li>Die intrinsischen und extrinsischen Kamerakalibrierungsparameter.</li>

<li>Die Matrizen von Velodyne zu Kamera.</li>

<li>Alle anderen "Sensor A" zu "Sensor B" Matrizen.</li>

<p>Diese Informationen stammen aus dieser Konfiguration:</p>
</ul>

<p float="left">
  <img src="https://gist.github.com/assets/125800618/71409e89-858b-4015-ad02-08d504d06361.png" width="1150" height="460" />

<ul>
<li>Velo-To-Cam (als V2C bezeichnet) enthält die Rotations- und Translationsmatrizen von der Velodyne zum linken kamera.</li>
<li>R0_rect unterstützt die Stereo Vision, indem es Bilder auf der gleichen Ebene ausrichtet.</li>
<li>P2 umfasst die Matrix, die nach der Kamerakalibrierung erhalten wurde, und enthält die intrinsische Matrix K sowie die extrinsischen Parameter.</li>
</ul>

#### Schritt 2.2: projizieren der LiDAR-Punkte in das Bild:

<ul>
<li>Die Punktes aus dem LiDAR-Rahmen in den Kamerarahmen werden projiziert.</p></li>
<li>Die projizierten Punktwolkenpunkte auf dem Bild werden visualisiert.</p></li>
</ul>

<p>Die Hauptformel, die verwendet werden soll, lautet wie folgt:</p>

<p>Y(2D) = P x R0 x R|t x X (3D)</p>

<p>Jedoch, wenn die Dimensionen betrachtet werden:</p>

<ul>
<li>P: [3x4]</li>
<li>R0: [3x3]</li>
<li>R|t = Velo2Cam: [3x4]</li>
<li>X: [3x1]</li>
</ul>

<p>Es müssen einige Punkte in homogene Koordinaten umgewandelt werden:</p>

<ul>
<li>RO muss sich ändern von 3x3 to 4x3</li>
<li>x mmuss sich ändern von 3x1 to 4x1</li>
</ul>

<p float="left">
<img src="https://gist.github.com/assets/125800618/64953cd2-4c52-42ff-a9be-f72a960f85c6.png" width="550" height="230" /> 
  
<img src="https://gist.github.com/assets/125800618/c2b4fa9a-2067-4f5e-8faf-f7e364f944ae.png" width="550" height="230" />

#### Hinweis:

<p>Die Matrizen stimmen nicht überein.</p>
<ul>
  <li>Die erste Operation, <code>P*R0</code>, wird aufgrund von <code>R0</code>, das nicht über 4 Zeilen verfügt, fehlschlagen.</li>
  <li><code>X</code> wird aufgrund der falschen Form nicht mit etwas multipliziert: Es soll transponiert werden, um <code>[3xn]</code> zu erhalten.</li>
  <li>Um alles zum Funktionieren zu bringen, müssen die Matrizen umgestaltet werden, und es muss eine andere Art von Koordinaten, die als homogene Koordinaten bezeichnet werden, angewendet werden.</li>
</ul>

#### <p> Homogene Koordinaten:</p>
<ul>
<p>Homogene Koordinaten werden verwendet, um Werte zu Matrizen hinzuzufügen und die Operationen zum Funktionieren zu bringen. Der Übergang von euklidischen zu homogenen Koordinaten erfolgt.</p>
</ul>

<img src="https://gist.github.com/assets/125800618/fca3ad7d-026c-48cf-a0af-a9ee47f59037.png" width="450" height="270" />

<ul>
<p>Die Operationen, mit denen gearbeitet wird, sind Translationen, Rotationen und manchmal sogar Skalierungen. Die Operationen im euklidischen Raum sind.</p>
</ul>

<p float="left">
  <img src="https://gist.github.com/assets/125800618/98acddec-ba0b-4f15-a64e-fb13f396f218.png" width="450" height="270" />


<p float="left">
  <img src="https://gist.github.com/assets/125800618/622dcc1e-9cab-4eaf-95e0-ae7aabc43c24.png" width="450" height="330" />

<ul>
<p>Es wird an den Matrizen gearbeitet, bei denen die Form nicht übereinstimmt, und es ergibt sich Folgendes.</p>
</ul>

<p float="left">
  <img src="https://gist.github.com/assets/125800618/7144a5cf-b8be-4762-b179-30148d3967ad.png" width="550" height="230" />

#### <p>Rückkehr zum euklidischen Raum:</p>
<ul>
Sobald die Multiplikationen durchgeführt wurden, soll zum euklidischen Raum zurückgekehrt werden, indem durch diese letzte Zahl geteilt wird.
</ul>

<p float="left">
  <img src="https://gist.github.com/assets/125800618/199c4132-3727-4811-b9b5-e7b361cef1db.png" width="480" height="230" />
</ul>

#### Visual Ergebnisse:

<p float="left">
  <img src="https://gist.github.com/assets/125800618/de3f172d-9016-433f-beb3-e48de9165620.png" width="500" height="230" />

<p float="left">
  <img src="https://gist.github.com/assets/125800618/e456c02f-b523-47b3-9ca4-982efdee6c8a.png" width="500" height="230" />

<p float="left">
  <img src="https://gist.github.com/assets/125800618/d54bad90-34fd-4753-8fcd-5f5877add960.png" width="500" height="230" />


#### Schritt 3: Objekterkennung mit YOLO

<p>Objekterkennung in 2D erkennen:</p>

#### Installation:
```bash
!python3 -m pip install Yolov4==2.0.2 # After Checking, YOLO 2.0.2 works without modifying anything. Otherwise keep 1.2.1
```
<ul>
<li>Das YOLOv4-Modell für die Objekterkennung wird geladen.</li>
<li>Die Objekterkennung mit dem YOLO-Modell auf dem Bild wird ausgeführt und die Ergebnisse werden angezeigt.</li>
</ul>

#### Visual Ergebnisse:

<p float="left">
  <img src="https://gist.github.com/assets/125800618/b86963f9-0dbd-4c96-a960-fec1b6b8d2c7.png" width="500" height="230" />
<p float="left">
  <img src="https://gist.github.com/assets/125800618/b9b69c54-9124-4109-ab9a-e4c7cc96974c.png" width="500" height="230" />

#### Schritt 4: Filterung von Ausreißern und Berechnung der Entfernungen zu den erkannten Objekten:

<p>Die Technik, die dafür verwendet werden kann, ist ein Verkleinerungsfaktor. Anstatt die gesamte Boudingbox zu betrachten, wird nur ein Teil davon betrachtet. Eine gängige Wahl ist eine Verkleinerung um 10-15%.</p>

<p>Berechnung der Entfernungen zu den erkannten Objekten.( Die Entfernung zu erkannten Objekten wird unter Verwendung der x-Komponente des LiDAR-Punkts berechnet, was eine Manhattan-Distanz entlang der x-Dimension des Ego-Autos darstellt.)</p>

#### Final Visual Ergebnisse:
<p float="left">
  <img src="https://gist.github.com/assets/125800618/5fbae6a8-1004-4301-b658-617c0b6ceb9e.png" width="500" height="230" />

#### Erstellen einer Daten-Pipeline:

<p>Eine Pipeline wird erstellt, die für jedes Bildpaar aus Kamera und LiDAR folgende Schritte durchführt:</p>

<ul>
<li>Anzeigen der LiDAR-Daten auf dem Kamerabild.</li>
<li>Objekterkennung in 2D.</li>
<li>Fusion von LiDAR-Punktwolken und Bounding-Boxen.</li>
</ul>

#### Video erstellen:

<ul>
<li>Erstellt ein Video, das den Prozess der Objekterkennung und Fusion von LiDAR- und Kameradaten visualisiert.</li>
<li>Lädt Bilddateien und Punktwolken, wendet die Fusion auf jedes Frame an und speichert das resultierende Video im Ausgabeverzeichnis.</li>
</ul>

  ![ezgif com-optimize](https://gist.github.com/assets/125800618/c5b35005-030d-4d39-a6e0-cfc8fb9870ff.gif)