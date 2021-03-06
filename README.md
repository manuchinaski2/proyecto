El proyecto consiste en la creación de un repositorio para la gestión colaborativa de proyecto java.

Se crea un fichero .gitignore con el siguiente contenido:
........................................................
.~
*.class
........................................................

Comprobando que funciona:
........................................................
usuario@usuario-System-Product-Name:~/manolo/proyecto$ git add *

The following paths are ignored by one of your .gitignore files:
Uno.class
Use -f if you really want to add them.
fatal: no files added
.......................................................

La intención es que no sean gestionados ficheros procedentes de copias de seguriad de archivos fuentes de java por el editor, ni los procedentes de la compilación de los mismos.

El repositorio consta de dos ramas tal y como muestra la siguiente salida:

............................................................
usuario@usuario-System-Product-Name:~/manolo/git$ git branch 
  desarrollo
* master
............................................................
La rama desarrollo es empleada para la eleboración del código, sin especificaciones algunas.
Mientrás que la rama master debe cumplir los siguientes requisitos:

1. El fichero debe tener una cabecera javadoc con el siguiente formato:
/********
*
*@author autor del proyecto
*@version 1.2
*/
Es decir debe contener el comentario con la sintáxis de javadac, además de contener las etiquetas author y version, conteniendo al menos el nombre del autor y una version en formato digito.digito.

2. Debe compilar correctamente, es decir la salida del comando javadoc ($?) debe ser cero.

3. Siempre que se haga "commit" debe borrar los fichero de compilación.

Todo esto se controla mediante "hooks", mas concretamente en el fichero pre-commit siguiente:

..............................................................................................
BRANCH=`git branch | grep \* |cut -c3-` ;
if [ $BRANCH != "master" ] ; then #acciones solo para rama master
        exit 0;
fi
for FILE in `git diff-index --name-status HEAD -- | cut -c3-` ; do
        egrep '^\/\*\*' $FILE 1>/dev/null; #comprobación sintáxis javado
        if [ $? != 0 ] ; then
                echo "Debes definir cabecera de javadoc en el fichero $FILE"
                exit 1
        fi
        egrep '@author [A-Za-z]+' $FILE 1>/dev/null; #comprobación etiqueta @author
        if [ $? != 0 ] ; then
                echo "Debes definir etiqueta author en javadoc en el fichero $FILE"
                exit 1
        fi
        egrep '@version [0-9]\.[0-9]' $FILE 1>/dev/null; #comprobación etiqueta @version
        if [ $? != 0 ] ; then
                echo "Debes definir etiqueta version en javadoc en el fichero $FILE"
                exit 1
        fi
        `javac Prueba.java 2>/dev/null`; #compilación de archivos
        if [ $? != 0 ] ; then
                echo "El fichero $FILE no compila correctamente"
                exit 1
        fi

done
rm *.class 2>/dev/null #borrado ficheros de compilación
exit 0;
..........................................................................................

Ejemplo de funcionamiento:

Creamos el fichero Nuevo.java:
.......................................................
public class Nuevo{
}
......................................................

La salida del git commit es:

............................................................
usuario@usuario-System-Product-Name:~/manolo/git$ git commit -m "Probando"
Debes definir cabecera de javadoc en el fichero Nuevo.java
............................................................

Lo tratamos en la rama desarrollo y lo modificamos y compilamos:

..................................................
/**
*@author Manuel
*@version 1.1
*/
public class Nuevo{
}
..................................................

Lo preparamos con git add y git commit:

..................................................
The following paths are ignored by one of your .gitignore files:
Nuevo.class
Use -f if you really want to add them.
fatal: no files added

usuario@usuario-System-Product-Name:~/manolo/git$ git commit -m "modificado"
[desarrollo 7ab813a] modificado
 Committer: usuario <usuario@usuario-System-Product-Name.(none)>

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 1 files changed, 2 insertions(+), 0 deletions(-)
 create mode 100644 Nuevo.java
...................................................

Y finalmente fusionamos las ramas:

...................................................

Cambiamos a la rama master
usuario@usuario-System-Product-Name:~/manolo/git$ git merge desarrollo 

...................................
Updating e59ca19..477abef
Fast-forward
 Uno.java |    1 +
 1 file changed, 1 insertion(+)
...................................

Y ya está preparado para el push a gitHub
