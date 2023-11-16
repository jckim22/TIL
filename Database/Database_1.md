# 요구사항(회사 테이블)
![](https://velog.velcdn.com/images/jckim22/post/53b2f615-4e52-4e5c-9db3-f32cf6eafd30/image.png)
![](https://velog.velcdn.com/images/jckim22/post/7352d70b-e75e-4052-828a-4fd8537b2553/image.png)



# 요구사항 정리

# Entity

![](https://velog.velcdn.com/images/jckim22/post/2c1c1097-5a3f-47a2-ad2c-c99a4c3f0231/image.png)

>부양가족은 이름을 기본키로 갖지만 약한개체이기 때문에 사원의 외래키와 부분키를 합쳐서 기본키로 사용해야한다.

# Relation



![](https://velog.velcdn.com/images/jckim22/post/e33d1b64-700d-4c30-8869-fb609b55681e/image.png)


>1. 여러 사원이 여러 프로젝트에서 일한다
2. 한 사원이 한 프로젝트를 관리한다.
3. 여러 사원이 하나의 부서에 속한다
4. 하나의 부품에 여러 부품이 속할 수 있다.
5. 여러 공급자가 여러 부품을 공급할 수 있다.
6. 여러 부품은 여러 프로젝트에 공급될 수 있다.
7. 하나의 사원에 여러 부양가족이 속한다.


# ERD

![](https://velog.velcdn.com/images/jckim22/post/e8856dbc-c914-4a3d-b60c-e5d2247d9146/image.png)

>다대다 관계를 해결하기 위해 새로운 관계 릴레이션을 생성했다.

