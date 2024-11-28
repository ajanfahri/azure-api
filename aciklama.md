# Azure DevOps API Entegrasyonu için JavaScript'te Erişim Belirteçlerini Güvenli Bir Şekilde Kullanma

Azure DevOps API'sini JavaScript projenize entegre etmek, iş akışlarını otomatikleştirmek ve depo bilgilerine erişmek için güçlü bir yol olabilir. Ancak, erişim belirteçlerini güvenli bir şekilde kullanmak, yetkisiz erişimi ve potansiyel güvenlik ihlallerini önlemek için çok önemlidir. Bu makale, bir JavaScript projesinde erişim belirteçlerini güvenli bir şekilde kullanma sürecinde size rehberlik edecektir.

## Azure DevOps'ta Kişisel Erişim Belirteci (PAT) Oluşturma

Öncelikle, Azure DevOps'ta bir Kişisel Erişim Belirteci (PAT) oluşturmanız gerekmektedir:

1. [Azure DevOps portalına](https://dev.azure.com/) gidin.
2. Sağ üst köşedeki kullanıcı profilinize tıklayın ve `Güvenlik` seçeneğini seçin.
3. `Kişisel Erişim Belirteçleri` altında, `Yeni Belirteç` butonuna tıklayın.
4. Belirtecinize bir ad verin, son kullanma tarihini belirleyin ve gerekli kapsamları seçin (örneğin, `Kod (Okuma)`).
5. `Oluştur` butonuna tıklayın ve belirteci kopyalayın. **Bu belirteci güvenli bir yerde saklayın ve paylaşmayın.**

## Neden Access Token Güvenli Kullanmalıyız?

Access Tokenlari, Azure DevOps kaynaklarınıza erişim sağlayan hassas bilgilerdir. Bu belirteçler ifşa edilirse, kötü niyetli kullanıcılar depolarınıza, boru hatlarınıza ve diğer kaynaklarınıza yetkisiz erişim sağlayabilir. Bu nedenle, belirteçleri güvenli bir şekilde kullanmak ve doğrudan kaynak kodunuza hardcode etmemek çok önemlidir.

## Adım Adım Rehber

### 1. ACCESS TOKEN Güvenli Bir Şekilde Saklama

Tokeni JavaScript dosyalarınıza hardcode etmek yerine, belirteci Git gibi sürüm kontrol sistemleri tarafından izlenmeyen ayrı bir yapılandırma dosyasında saklayın.

1. Proje dizininizde bir `config.js` dosyası oluşturun:
    ```javascript
    // config.js
    const config = {
        organization: 'your-organization-name',
        project: 'your-project-name',
        personalAccessToken: 'YOUR_PERSONAL_ACCESS_TOKEN'
    };

    export default config;
    ```

2. `config.js` dosyasını `.gitignore` dosyanıza ekleyin, böylece Git tarafından izlenmez:
    ```plaintext
    config.js
    ```

### 2. Belirteci JavaScript Kodunuzda Kullanma

Yapılandırma dosyasını içe aktarın ve belirteci API isteklerinizde kullanın:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Azure DevOps Repo Bilgileri</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #f4f4f4;
        }
        .container {
            background-color: #fff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            max-width: 600px;
            width: 100%;
        }
        h1 {
            text-align: center;
            color: #333;
        }
        .repo-list, .branch-list, .branch-details {
            margin-top: 20px;
        }
        .repo-list p, .branch-list p {
            margin: 10px 0;
            color: #555;
            cursor: pointer;
        }
        .repo-list p:hover, .branch-list p:hover {
            text-decoration: underline;
        }
        .back-button {
            margin-top: 20px;
            cursor: pointer;
            color: #007bff;
            text-decoration: none;
        }
        .back-button:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Azure DevOps Repo Bilgileri</h1>
        <div id="repo-list" class="repo-list"></div>
        <div id="branch-list" class="branch-list" style="display: none;"></div>
        <div id="branch-details" class="branch-details" style="display: none;"></div>
        <a id="back-button" class="back-button" style="display: none;" onclick="goBack()">Geri Dön</a>
    </div>

    <script type="module">
        import config from '../config.js';
        const organization = config.organization;
        const project = config.project;
        const personalAccessToken = config.personalAccessToken;
        
        const fetchRepoInfo = async () => {
            const response = await fetch(`https://dev.azure.com/${organization}/${project}/_apis/git/repositories?api-version=7.1`, {
                headers: {
                    'Authorization': 'Basic ' + btoa(':' + personalAccessToken)
                }
            });
            const data = await response.json();
            const repoListDiv = document.getElementById('repo-list');
            repoListDiv.innerHTML = '<h2>Repo Listesi</h2>';
            data.value.forEach(repo => {
                const repoElement = document.createElement('p');
                console.log(repo.id)
                repoElement.textContent = repo.name;
                repoElement.onclick = () => fetchBranchInfo(repo.id);
                repoListDiv.appendChild(repoElement);
            });
        };

        const fetchBranchInfo = async (repoId) => {
            const response = await fetch(`https://dev.azure.com/${organization}/${project}/_apis/git/repositories/${repoId}/refs?filter=heads/&api-version=7.1`, {
                headers: {
                    'Authorization': 'Basic ' + btoa(':' + personalAccessToken)
                }
            });
            const data = await response.json();
            const branchListDiv = document.getElementById('branch-list');
            branchListDiv.innerHTML = '<h2>Branch Listesi</h2>';
            data.value.forEach(branch => {
                const branchElement = document.createElement('p');
                const cleanBranchName = branch.name.replace("refs/heads/", "");
                branchElement.textContent = cleanBranchName;
                
                branchElement.onclick = () => fetchBranchDetails(repoId, cleanBranchName);
                branchListDiv.appendChild(branchElement);
            });
            document.getElementById('repo-list').style.display = 'none';
            branchListDiv.style.display = 'block';
            document.getElementById('back-button').style.display = 'block';
        };

        const fetchBranchDetails = async (repoId, branchName) => {
            const response = await fetch(`https://dev.azure.com/${organization}/${project}/_apis/git/repositories/${repoId}/refs?filter=heads/${branchName}&api-version=7.1`, {
                headers: {
                    'Authorization': 'Basic ' + btoa(':' + personalAccessToken)
                }
            });
            const data = await response.json();
            const branchDetailsDiv = document.getElementById('branch-details');
            branchDetailsDiv.innerHTML = '<h2>Branch Detayları</h2>';
            
            if (data.value.length > 0) {
                const branch = data.value[0];
                console.log(branch)
                branchDetailsDiv.innerHTML += `<p><strong>Branch Adı:</strong> ${branch.name}</p>`;
                branchDetailsDiv.innerHTML += `<p><strong>Branch URL:</strong> <a href="${branch.url}" target="_blank">${branch.url}</a></p>`;
                branchDetailsDiv.innerHTML += `<p><strong>Son Güncelleme:</strong> ${new Date(branch.date).toLocaleString()}</p>`;
            } else {
                branchDetailsDiv.innerHTML += `<p>Branch detayları bulunamadı.</p>`;
            }
            document.getElementById('branch-list').style.display = 'none';
            branchDetailsDiv.style.display = 'block';
        };

        const goBack = () => {
            document.getElementById('branch-details').style.display = 'none';
            document.getElementById('branch-list').style.display = 'none';
            document.getElementById('repo-list').style.display = 'block';
            document.getElementById('back-button').style.display = 'none';
        };

        window.goBack = goBack;
        
        fetchRepoInfo();
    </script>
</body>
</html>
