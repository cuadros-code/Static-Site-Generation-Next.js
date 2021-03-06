# Static Site Generation SSG
`getStaticProps` solo se ejecuta en build time y en el servidor

 You should use getStaticProps when:
- The data required to render the page is available at build time ahead of a user’s request.
- The data comes from a headless CMS.
- The data can be publicly cached (not user-specific).
- The page must be pre-rendered (for SEO) and be very fast — getStaticProps generates HTML and JSON files, both of which can be cached by a CDN for performance.

```
  export const getStaticProps: GetStaticProps = async (ctx) => {

    const { name } = ctx.params as { name: string };

    const { data } = await pokeApi.get<PokemonDetail>(`/pokemon/${name}`);

    return {
      props: {
        pokemon: data
      }
    }
  }
```

<hr>

### getStaticPaths
- `params` es las rutas que se aceptam
- `fallback` en caso de que sea `false` y la pagina no existe retorna un 404, en caso de `blocking` si renderiza el componente
```
  export const getStaticPaths: GetStaticPaths = async (ctx) => {
    return {
      paths: [
        {
          params: {
            id: '1',
          }
        }
      ],
      // fallback: "blocking"
      fallback: false
    }
  }
```

<hr>

### Incremental Static Regeneration

- regenerate static pages every n seconds
- add in return the property <span style='color: yellow'>revalidate: [seconds]</span>

```
  export const getStaticProps: GetStaticProps = async (ctx) => {

    const { name } = ctx.params as { name: string };

    const { data } = await pokeApi.get<PokemonDetail>(`/pokemon/${name}`);

    return {
      props: {
        pokemon: data
      },
      revalidate: 86400 // 1 day
    }
  }
```


### Incremental Static Generation
- generate content after run build
- change the property `fallback` from `false` to `blocking`

```
  export const getStaticPaths: GetStaticPaths = async (ctx) => {

    const pokemons = [...Array(151)].map( (_, i) => `${i + 1}` );

    return {
      paths: pokemons.map( (id) => ({
        params: {
          id
        }
      })),
      fallback: 'blocking'
    }
  }
```
- Then validate if the request data exist 

```
  export const getStaticProps: GetStaticProps = async ({ params }) => {
    
    const { id } = params as { id: string };

    const data  = await getPokemonInfo(id)

    if( !data ) {
      return {
        redirect: {
          destination: '/',
          permanent: false
        }
      }
    }

    return {
      props: {
        pokemon: data
      },
      revalidate: 86400 // 1 day
    }
  }
```