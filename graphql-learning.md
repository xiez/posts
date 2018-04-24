## GraphQL vs REST

### REST 缺点 ###

* Backend dev

 每个资源定义一个endpoint，多个资源需要在多个endpoint返回

 ```
 /groups/
 /groups/:id/
 /groups/:id/repos/
 /groups/:id/repos/:id/
 ```

 如果一个endpoint返回多个不相关资源，会导致跟客户端耦合，增加代码复杂度

 [https://dev.seafile.com/seahub/#groups/](https://dev.seafile.com/seahub/#groups/)

 ```
 /groups/?with_repos=1
 ```

 接口变动需要升级版本

 /api2/shared-links/

 ```
 {
  "fileshares": [
    {
      "username": "user@example.com",
      "repo_id": "a582d3bc-bcf5-421e-9125-741fa56d18d4",
      "ctime": null,
      "s_type": "d",
      "token": "e410827494",
      "view_cnt": 0,
      "path": "\/123\/"
    },

    ....

  ]
 }
 ```

 /api/v2.1/share-links/

 ```
 [
    {
        "username": "lian@lian.com",
        "repo_id": "c474a093-19dc-4ddf-b0b0-72b33214ba33",
        "ctime": "2017-04-01T02:35:57+00:00",
        "expire_date": "",
        "token": "6afa667ff2c248378b70",
        "view_cnt": 0,
        "link": "https://cloud.seafile.com/d/6afa667ff2c248378b70/",
        "obj_name": "/",
        "path": "/",
        "is_dir": true,
        "is_expired": false,
        "repo_name": "seacloud.cc.124"
    },

    ...

 ]
 ```

* Frontend dev

 Under-fetch: 某个endpoint返回的字段不够，需要请求多个endpoint

 [https://dev.seafile.com/seahub/#groups/](https://dev.seafile.com/seahub/#groups/)

 Over-fetch: 一个endpoint返回的字段过多，只需要某些字段，影响性能／带宽

 [https://dev.seafile.com/seahub/#my-libs/](https://dev.seafile.com/seahub/#my-libs/)

 ```
 [
 {
    "root": "",
    "modifier_email": "xiez1989@gmail.com",
    *"name": "test",
    "permission": "rw",
    *"size_formatted": "25\u00a0bytes",
    "virtual": false,
    *"mtime_relative": "<time datetime=\"2018-04-14T14:38:03\" is=\"relative-time\" title=\"Sat, 14 Apr 2018 14:38:03 +0800\" >4 days ago<\/time>",
    "head_commit_id": "88fefc7af914e1c0cdddb0be8cd1c4aba514f35b",
    *"encrypted": false,
    "version": 1,
    *"mtime": 1523687883,
    "owner": "xiez1989@gmail.com",
    "modifier_contact_email": "xiez1989@gmail.com",
    "type": "repo",
    *"id": "d7d22ac3-696d-468b-8f62-644547e939f4",
    "modifier_name": "\u90d1\u52f0\ud504\ub85c\ud30c fran\u00e7ais",
    *"size": 25
  },

  ...

 ]
 ```

 返回结果由 API 开发者控制，需要对结果进行处理，影响开发效率

* User

 移动设备并发请求有限制，或网速不快，多余的网络请求会降低用户体验

### GraphQL ###

* Schema Definition Language (SDL)

  [https://graphql.org/learn/schema/](https://graphql.org/learn/schema/)

  ```jvascript
  type Group {
    id: Int!
    name: String!
    createAt: DateTime!

    repos: [Repo]
  }

  type Repo {
    id: String!
    name: String!
    owner: String!
    size: Int!

  }
  ```



  - Strong type


* Query and Mutation

  [https://graphql.org/learn/queries/](https://graphql.org/learn/queries/)

  List users with count users:

  ```
  query UsersAndCount {
    usersCnt
    users {
      objects {
        username
        email
        jointime: dateJoined
      }
    }
  }
  ```

  Result:

  ```
  {
    "data": {
      "usersCnt": 8922,
      "users": {
        "objects": [
          {
            "username": "seafile2012",
            "email": "seafile2012@gmail.com",
            "jointime": "2016-10-13T04:13:45+00:00"
          },
          {
            "username": "xiez",
            "email": "xiez1989@gmail.com",
            "jointime": "2016-10-13T05:38:30+00:00"
          },
          {
            "username": "daniel.pan",
            "email": "daniel.pan@seafile.com",
            "jointime": "2016-10-13T08:05:49+00:00"
          },
          {
            "username": "authen-tic",
            "email": "mbeaudet@authen-tic.ca",
            "jointime": "2016-10-13T11:34:42+00:00"
          },
          {
            "username": "dejand",
            "email": "dejan@ld.si",
            "jointime": "2016-10-14T11:52:08+00:00"
          }
        ]
      }
    }
  }
  ```


 - SQL:

   ```
   mysql> select user as email, nickname, contact_email from profile_profile;
   +--------------------------------------------+---------------------------+------------------------+
   | email                                      | nickname                  | contact_email          |
   +--------------------------------------------+---------------------------+------------------------+
   | freeplant@163.com                          |  潘凌涛                   | NULL                   |
   | lingjun.li1@gmail.com                      |                           | NULL                   |
   | mysnowls@163.com                           | Shuai Lin                 | NULL                   |
   | xiez1989@gmail.com                         | 郑勰프로파 français       | NULL                   |
   | imwhatiam123@gmail.com                     | Vincent Lee               | imwhatiam123@gmail.com |
   ```


* [GraphIQL](https://github.com/graphql/graphiql)

  [https://customer.seafile.com/admin/gql](https://customer.seafile.com/admin/gql)

* Architecture

  [https://www.howtographql.com/basics/3-big-picture/](https://www.howtographql.com/basics/3-big-picture/)

  ![GraphQL Architecture](https://i.imgur.com/oOVYriG.png "GraphQL Architecture")


* New develop flow for frontend and backend

  [Prisma (Backend-as-a-service)](https://www.prisma.io/)

## GraphQL Client

Why?

[howtographql](https://www.howtographql.com/basics/3-big-picture/)

> When fetching data from a REST API, most applications will have to go through the following steps:
>  
>     construct and send HTTP request (e.g. with fetch in Javascript)
>     receive and parse server response
>     store data locally (either simply in memory or persistent)
>     display data in the UI
>  
> With the ideal declarative data fetching approach, a client shouldn’t be doing more than the following two steps:
>  
>     describe data requirements
>     display data in UI
>  
> All the lower-level networking tasks as well as storing the data should be abstracted away and the declaration of data dependencies should be the dominant part.


Fetch API:

```
      const url = `${apiEndpoint}?page=${state.page+1}&pageSize=${state.pageSize}&sorted=${sort_patt}`;
      fetch(url, {credentials: 'same-origin'})
        .then(res => res.json())
        .then(res => {
          // Now just get the rows of data to your React Table (and update anything else like total pages or loading)
          this.setState({
            data: res.data,
            pages: res.page_nums,
            loading: false
          });
        });
```
### Apollo

- Declarative data fetching

```
export const USERS_QUERY = gql`
query  UsersQuery($page: Int, $pageSize: Int, $sortby: String){
  users(page: $page, pageSize: $pageSize, sortby: $sortby){
    pages
    hasNext
    hasPrev
    objects {
      id
      username
      email
      firstname: firstName
      lastname: lastName
      active: isActive
      dateJoined
      country
      company
    }
  }
}
`;

    client.query({
      query: USERS_QUERY,
      variables: {
        pageSize: pageSize,
        page: page+1,
        sortby: sort_patt
      },
    })
      .then(({ data: { users } }) => {
        console.log('finish fetch data, result: ', users);
        return users;
      })
      .then(users => {
        this.setState({
          data: users.objects,
          pages: users.pages,
          loading: false
        });
      })
      .catch(() => console.log('error in fetch data'));
```

- Cache

  - Cache update

     - Refetch

     - Polling

     - Subscription

  Refer [Apollo dev blog](https://dev-blog.apollodata.com/the-concepts-of-graphql-bc68bd819be3) for details.

- Network layer

  ```
  const client = new ApolloClient({
  link: new HttpLink({
    uri: '/admin/gql',
    credentials: 'same-origin'
  }),
  cache: new InMemoryCache(),
  });
  ```

  - Apollo Link

  - Middleware

### Relay

[Relay Classic vs Relay Modern](https://facebook.github.io/relay/docs/en/new-in-relay-modern.html)

* 学习曲线过高，不适合入门

## GraphQL Server

### Graphene & Django Graphene

### Performance & Security

[https://www.howtographql.com/advanced/4-security/](https://www.howtographql.com/advanced/4-security/)

```
query IAmEvil {
  author(id: "abc") {
    posts {
      author {
        posts {
          author {
            posts {
              author {
                # that could go on as deep as the client wants!
              }
            }
          }
        }
      }
    }
  }
}
```

 - Timtout

 - Maximum Query Depth

 - Throttling


