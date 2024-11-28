# Azure DevOps Repo Bilgileri

Bu proje, Azure DevOps API'sini kullanarak bir organizasyondaki repo ve branch bilgilerini görüntüleyen basit bir web arayüzü sağlar. Kullanıcılar, repo isimlerine tıklayarak ilgili branch ve detay bilgilerini görebilirler.

## Özellikler

- Azure DevOps API'si ile repo bilgilerini çekme
- Repo isimlerini listeleme
- Branch isimlerini listeleme
- Branch detaylarını görüntüleme
- Geri dönme işlevselliği

## Gereksinimler

- Azure DevOps hesabı
- Kişisel Erişim Belirteci (PAT) ile Azure DevOps API erişimi
- Web tarayıcısı

## Kurulum

1. Bu projeyi yerel makinenize klonlayın veya indirin.
2. `index.html` dosyasını açın ve aşağıdaki değişkenleri kendi bilgilerinize göre güncelleyin:
    ```javascript
    const organization = 'your-organization-name'; // Organizasyon adınızı buraya yazın
    const project = 'your-project-name'; // Proje adınızı buraya yazın
    const personalAccessToken = 'YOUR_PERSONAL_ACCESS_TOKEN'; // Kişisel erişim belirtecinizi buraya yazın
    ```

## Kullanım

1. `index.html` dosyasını bir web tarayıcısında açın.
2. Repo isimleri listelenecektir. Bir repo ismine tıklayarak ilgili branch isimlerini görebilirsiniz.
3. Bir branch ismine tıklayarak branch detaylarını görüntüleyebilirsiniz.
4. "Geri Dön" butonuna tıklayarak önceki sayfaya dönebilirsiniz.

## Örnek Kod

### HTML ve CSS
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
