# Auto-contrast
Ну крч, нам по сучему ЭВМу задали написать прогу, которая будет автоматически делать норм контрастность у фоток формата .ppm/.pgm

Ахуеть да?

Ну в общем  вы в argv[1] передаете количество потоков (это типо было дз на распараллеливание...), в argv[2] передаете название файла .ppm/.pgm, в argv[3] передаете название нового файла
, а в argv[4] коэффициент (это чтобы он скипал единичные очень светлые и очень темные пиксели при растягивании диапазона)