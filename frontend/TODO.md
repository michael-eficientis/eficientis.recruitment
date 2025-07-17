<p align="center">
  <a href="https://eficientis.app">
    <img src="https://eficientis.app/assets/eficientis.isotipo.png" width="90px" />
  </a>
</p>

<h1 align="center">
  Eficientis Frontend Challenge
</h1>

This technical assessment requires the development of a client-side application that integrates with GitHub's GraphQL API v4 to display user repository information. The application should demonstrate proficiency in modern Angular development practices and efficient API integration.

## Technical Requirements

1. **Framework & UI**

   - Angular (latest LTS version)
   - Ng-Zorro Ant Design or Angular Material for UI components

2. **Development Tools**

   - TypeScript (latest stable version)
   - RxJS for reactive programming
   - Angular CLI for project management
   - ESLint for code linting
   - Prettier for code formatting
   - Jest for unit testing
   - Cypress for E2E testing
   - Apollo Client for GraphQL integration

3. **State Management**

   - NgRx for global state management
   - NgRx Effects for side effects handling
   - NgRx Store DevTools for debugging

4. **API Integration**
   - Angular HttpClient for API calls
   - RxJS operators for data transformation
   - Proper error handling and loading states
   - GraphQL client integration with Apollo Client

## User Stories

1. **As a User**
   - I want to search for a GitHub user by username
   - I want to view the user's profile information (avatar, name, bio, followers, following)
   - I want to see a paginated list of the user's repositories ordered by star count
   - I want to be able to sort repositories by different criteria (stars, forks, updated date)
   - I want to view detailed repository information including:
     - Repository name and description
     - Number of stars, forks, and watchers
     - Primary programming language
     - Last updated date
     - Link to GitHub repository page
     - Repository license information (if available)
     - Repository topics (if any)

## Implementation Requirements

1. **UI/UX**

   - Responsive layout for devices with minimum resolution of 320x480
   - Modern and clean design using Ng-Zorro or Angular Material
   - Proper loading states with spinners
   - Error handling with user-friendly messages
   - Pagination for repository lists with infinite scroll
   - Repository sorting options
   - Repository filtering options

2. **Code Structure**

   - Feature-based architecture
   - Lazy-loaded feature modules
   - Shared services and components
   - Proper error boundaries
   - Clean and maintainable code with proper comments

3. **Performance**
   - Efficient GraphQL queries with pagination
   - Image lazy loading
   - Component-level caching
   - Optimized bundle size with tree-shaking
   - Proper use of Angular's change detection strategy

## Submission Guidelines

1. Fork this repository to your GitHub account
2. Complete the implementation
3. Ensure all tests pass
4. Document your solution in `SOLUTION.md`
5. Submit the pull request with your changes

The application must be fully functional in a local development environment. No deployment is required.

For any questions regarding the assessment, please submit an issue in the repository.

## API Integration Details

### 🔐 Autenticación

1. **Personal Access Token (PAT)**

   - Genera un [token personal desde GitHub](https://github.com/settings/tokens) con el scope `public_repo` o `repo` según lo necesites.
   - Usa el token para autenticación HTTP en los headers:

     ```http
     Authorization: Bearer YOUR_PERSONAL_ACCESS_TOKEN
     ```

   - Almacena el token en variables de entorno (ej. `environment.ts`) **y nunca lo expongas en el frontend.**

2. **Rate Limiting**

   - GitHub impone límites de uso basados en el usuario autenticado.
   - Monitorea `rateLimit` en cada respuesta para controlar el consumo:

     ```graphql
     query {
     	rateLimit {
     		remaining
     		resetAt
     	}
     }
     ```

   - Implementa:

     - Mensajes amigables cuando el límite se alcance
     - Reintentos con **exponential backoff** si es transitorio

### GraphQL Queries

#### Buscar usuario

```graphql
query SearchUser($query: String!, $first: Int!) {
	search(query: $query, type: USER, first: $first) {
		userCount
		pageInfo {
			hasNextPage
			endCursor
		}
		edges {
			node {
				... on User {
					login
					name
					avatarUrl
					bio
					followers {
						totalCount
					}
					following {
						totalCount
					}
					repositories(
						first: 10
						orderBy: { field: STARGAZERS, direction: DESC }
					) {
						totalCount
						nodes {
							name
							stargazerCount
						}
					}
				}
			}
		}
	}
}
```

> ⚠️ Límite máximo de `first: 100`. No se pueden obtener más de \~1000 resultados usando `search`.

```graphql
query SearchUser($query: String!, $first: Int!) {
	search(query: $query, type: USER, first: $first) {
		userCount
		pageInfo {
			hasNextPage
			endCursor
		}
		edges {
			node {
				... on User {
					login
					name
					avatarUrl
					bio
					followers {
						totalCount
					}
					following {
						totalCount
					}
					repositories(
						first: 10
						orderBy: { field: STARGAZERS, direction: DESC }
					) {
						totalCount
						nodes {
							name
							stargazerCount
						}
					}
				}
			}
		}
	}
}
```

> ⚠️ Límite máximo de `first: 100`. No se pueden obtener más de \~1000 resultados usando `search`.

---

#### Obtener repositorios del usuario (paginado y ordenado)

```graphql
query UserRepositories(
	$login: String!
	$first: Int!
	$after: String
	$orderBy: RepositoryOrder
) {
	user(login: $login) {
		repositories(first: $first, after: $after, orderBy: $orderBy) {
			totalCount
			pageInfo {
				hasNextPage
				endCursor
			}
			nodes {
				name
				description
				stargazerCount
				forkCount
				watchers {
					totalCount
				}
				primaryLanguage {
					name
				}
				licenseInfo {
					name
					url
				}
				topics(first: 10) {
					nodes {
						name
					}
				}
				updatedAt
				url
			}
		}
	}
}
```

> ✅ `orderBy` puede ser `STARGAZERS`, `UPDATED_AT`, etc. Asegúrate de testear paginación con cada tipo de ordenamiento.

---

#### Detalles de un repositorio

```graphql
query RepositoryDetails($owner: String!, $name: String!) {
	repository(owner: $owner, name: $name) {
		name
		description
		stargazerCount
		forkCount
		watchers {
			totalCount
		}
		primaryLanguage {
			name
		}
		languages(first: 3, orderBy: { field: SIZE, direction: DESC }) {
			nodes {
				name
				color
			}
		}
		licenseInfo {
			name
			url
		}
		topics(first: 10) {
			nodes {
				name
			}
		}
		defaultBranchRef {
			target {
				... on Commit {
					history(first: 1) {
						edges {
							node {
								committedDate
							}
						}
					}
				}
			}
		}
		url
		homepageUrl
		createdAt
		updatedAt
	}
}
```

> ⚠️ Si `defaultBranchRef` es `null` (caso raro, pero posible), asegúrate de manejarlo para evitar errores de ejecución.

---

### ❗ Error Handling

1. **Errores del API**

   - Captura y muestra errores de red (`HttpErrorResponse`, `NetworkError`) con mensajes como:

     > "No se pudo conectar a GitHub. Revisa tu conexión o el token de acceso."

2. **Errores de GraphQL**

   - Usa la propiedad `errors` de la respuesta para validar campos faltantes, errores de permisos, etc.
   - En modo desarrollo, muestra trazas o mensajes técnicos.

3. **Rate Limiting**

   - Si `rateLimit.remaining === 0`, informa al usuario con el tiempo de espera (`resetAt`)
   - Opción avanzada: muestra una cuenta regresiva para reintento automático.

4. **Reintentos automáticos**

   - Para errores transitorios, aplica **exponential backoff**:

     ```ts
     retryWhen((errors) =>
     	errors.pipe(
     		scan((acc, error) => {
     			if (acc >= 3) throw error;
     			return acc + 1;
     		}, 0),
     		delay((acc) => 1000 * Math.pow(2, acc))
     	)
     );
     ```

## Live demo

https://ghsearch.lovable.app/
