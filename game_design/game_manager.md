---
layout: default
title: Home Page
---

# Game Manager

> Um gerenciador de um jogo deve ser o núcleo do jogo, funcionando como uma cola que une tudo o que é primordial no sistema de forma elegante, performática e legível. Aqui listo meu ponto de vista sobre como criar um Game Manager da forma que considero a melhor até o momento.

Todo jogo deve ter uma classe base de onde sairão heranças que formarão as entidades contidas na cena. Uma entidade é qualquer coisa colocada no jogo com um propósito de se comportar de forma distinta dos demais tipos de entidades. Como princípios básicos da Programação Orientada a Objetos, devemos ter uma classe base Entity, mais do que isso, toda entidade deve guardar dados comuns a todos os objetos, como a posição, rotação e escala do objeto, ou até mesmo os componentes (comportamentos) que serão adicionados nele em tempo de código (dev time) ou tempo de execução (run time).

As características relacionadas a posição, rotação e escala, tem em comum entre si a propriedade de ter 3 valores, um para o eixo X, um para o eixo Y e um para o eixo Z.

```java
public class Vector3 {

	public float x;
	public float y;
	public float z;

	public Vector3(float x, float y, float z){
		this.x = x;
		this.y = y;
		this.z = z;
	}

	public Vector3() {
		Vector3 vec = Vector3.zero();
		this.x = vec.x;
		this.y = vec.y;
		this.z = vec.z;
	}

	public static Vector3 zero(){
		return new Vector3(0f, 0f, 0f);
	}
}
```

A classe Transform deve guardar esses vetores.

```java
public class Transform {

	public Vector3 position;
	public Vector3 rotation;
	public Vector3 scale;

	public Transform(Vector3 position, Vector3 rotation, Vector3 scale){
		this.position = position;
		this.rotation = rotation;
		this.scale = scale;
	}

	public Transform(){
		this.position = Vector3.zero();
		this.rotation = Vector3.zero();
		this.scale = Vector3.zero();
	}
}
```

Agora que temos nossas classes que guardam as informações básicas de toda entidade, vamos definir a classe mãe de todos os comportamentos que poderemos adicionar às entidades.

```java
public abstract class Component {

    protected Entity entity;
    protected boolean isEnabled;

    public Component(Entity entity){
        this.entity = entity;
    }

    public abstract void update();
}
```

A classe Component deve ter conhecimento da entidade a qual ela irá alterar o comportamento, mas a entidade não deve conhecer a implementação de nenhum de seus comportamentos, isso se deve ao princípio da inversão de dependência, que
