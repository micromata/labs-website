---
title: Safari, iOS und Autoplay
author: abestvater
categories: [Quick Tips]
tags: [ios]
shortdesc: Wie WebKit mit Videos und autoplay umgeht.
featured: true
---

Hin und wieder möchte man ein Video im HTML einbinden und es  abspielen lassen ohne das der Benutzer es explizit startet. Verschiedene Browser haben diesbezüglich – dankenswerter Weise – gewisse  Einschränkungen. Es gibt jedoch Use Cases in denen es durchaus valide  ist diese Einschränkungen zu umgehen. Beispielsweise Animationen.

Selbststartende Videos sind mit dem `<video>`  Element, das wir mit HTML 5 serviert bekommen haben, zunächst mal  einfach umzusetzen. Wenn wir uns allerdings mit der Darstellung auf  mobilen Geräten beschäftigt, stellen wir schnell fest, dass es sich auf  dem iPhone anders verhält als erwartet. Da Apple alle Browser dazu  zwingt, die Render Engine WebKit zu verwenden (siehe [§2.5.6 der App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/#software-requirements)), verhalten sich Chrome, Firefox, Safari und alle anderen möglichen Browser, gleichermaßen unerwartet.

## Abspielen von Videos ohne User Interaction unter WebKit auf dem iPhone

Seit 2016 erlaubt WebKit das Abspielen von Videos auf iOS ohne  direkte Nutzerinteraktion – also ohne, dass der Nutzer das Video bewusst startet. Es gibt dazu also zwei Möglichkeiten: zum einen das  Autoplay-Attribut direkt am Video-Tag und zum anderen über die  play()-Methode der Web-Media-API. Dazu müssen wir allerdings einige  Dinge beachten, denn ganz so wie die Desktop-Version verhält sich WebKit auf dem iPhone nicht.

1. Das Video muss **stumm** sein! Das erreicht man bei Videos mit Tonspur mit dem **Muted-Attribut**. Bei Videos, die keine Tonspur enthalten wird das Attribute nicht  zwingend benötigt, da es als zwangsläufig stummes Video erkannt wird. Es ist allerdings trotzdem empfehlenswert, das Muted-Attribut zu setzen,  da ein Video, dass keinen Ton abspielt, trotzdem eine Tonspur haben  kann, die lediglich keinen Ton beinhaltet. Auch falls das Video mal  ausgetauscht wird, ist so sichergestellt, dass es stumm ist und  abgespielt werden kann.
2. Das Video muss **inline** abspielbar sein – also  abspielbar ohne dass es im Fullscreen angezeigt werden muss. Denn Videos werden (zum Glück!) nicht automatisch im Fullscreen geöffnet. Deshalb  muss dem Video-Element das **Playsinline-Attribut** gegeben werden, damit das Video auch direkt auf der Seite abgespielt werden kann.
3. Die dritte Bedingung gilt **nur für das Autoplay-Attribut**. Videos, die außerhalb des **Viewports** liegen, werden nicht abgespielt, bzw. pausiert sobald sie den Viewport  verlassen und wieder abgespielt, wenn sie wieder vollständig im Viewport liegen. Die play()-Methode funktioniert ungeachtet der Position des  Videos.
