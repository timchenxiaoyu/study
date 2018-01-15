
### 拉取镜像manifest
```go
	acceptMediaTypes := []string{schema1.MediaTypeManifest, schema2.MediaTypeManifest}
	digest, mediaType, payload, err := m.srcClient.PullManifest(tag, acceptMediaTypes)
```
tag 就是 reference
PullManifest实现
```go
func (r *Repository) PullManifest(reference string, acceptMediaTypes []string) (digest, mediaType string, payload []byte, err error) {
	req, err := http.NewRequest("GET", buildManifestURL(r.Endpoint.String(), r.Name, reference), nil)
	if err != nil {
		return
	}

	for _, mediaType := range acceptMediaTypes {
		req.Header.Add(http.CanonicalHeaderKey("Accept"), mediaType)
	}

	resp, err := r.client.Do(req)
	if err != nil {
		err = parseError(err)
		return
	}

	defer resp.Body.Close()
	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		return
	}

	if resp.StatusCode == http.StatusOK {
		digest = resp.Header.Get(http.CanonicalHeaderKey("Docker-Content-Digest"))
		mediaType = resp.Header.Get(http.CanonicalHeaderKey("Content-Type"))
		payload = b
		return
	}

	err = &registry_error.Error{
		StatusCode: resp.StatusCode,
		Detail:     string(b),
	}

	return
}

```
返回的头里面包含digest、mediaType和mainfest内容b。解析出来，

## 判读blob是否存在
```go
for _, blob := range blobs {
		exist, ok := m.blobsExistence[blob]
		if !ok {
			exist, err = m.dstClient.BlobExist(blob)
			if err != nil {
				m.logger.Errorf("an error occurred while checking existence of blob %s of %s:%s on %s: %v", blob, name, tag, m.dstURL, err)
				return "", err
			}
			m.blobsExistence[blob] = exist
		}

		if !exist {
			m.blobs = append(m.blobs, blob)
		} else {
			m.logger.Infof("blob %s of %s:%s already exists in %s", blob, name, tag, m.dstURL)
		}
	}
```

## 拉取并上传不存在的blob
```go
func (r *Repository) PullBlob(digest string) (size int64, data io.ReadCloser, err error) {
	req, err := http.NewRequest("GET", buildBlobURL(r.Endpoint.String(), r.Name, digest), nil)
	if err != nil {
		return
	}

	resp, err := r.client.Do(req)
	if err != nil {
		err = parseError(err)
		return
	}

	if resp.StatusCode == http.StatusOK {
		contengLength := resp.Header.Get(http.CanonicalHeaderKey("Content-Length"))
		size, err = strconv.ParseInt(contengLength, 10, 64)
		if err != nil {
			return
		}
		data = resp.Body
		return
	}
	// can not close the connect if the status code is 200
	defer resp.Body.Close()

	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		return
	}

	err = &registry_error.Error{
		StatusCode: resp.StatusCode,
		Detail:     string(b),
	}

	return
}

```
## 上传blob层
```go
func (r *Repository) PushBlob(digest string, size int64, data io.Reader) error {
	location, _, err := r.initiateBlobUpload(r.Name)
	if err != nil {
		return err
	}
	return r.monolithicBlobUpload(location, digest, size, data)
}

func (r *Repository) monolithicBlobUpload(location, digest string, size int64, data io.Reader) error {
	req, err := http.NewRequest("PUT", buildMonolithicBlobUploadURL(location, digest), data)
	if err != nil {
		return err
	}

	resp, err := r.client.Do(req)
	if err != nil {
		return parseError(err)
	}

	defer resp.Body.Close()

	if resp.StatusCode == http.StatusCreated {
		return nil
	}

	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		return err
	}

	return &registry_error.Error{
		StatusCode: resp.StatusCode,
		Detail:     string(b),
	}
}
```
## 上传manifest
```go
func (r *Repository) PushManifest(reference, mediaType string, payload []byte) (digest string, err error) {
	req, err := http.NewRequest("PUT", buildManifestURL(r.Endpoint.String(), r.Name, reference),
		bytes.NewReader(payload))
	if err != nil {
		return
	}
	req.Header.Set(http.CanonicalHeaderKey("Content-Type"), mediaType)

	resp, err := r.client.Do(req)
	if err != nil {
		err = parseError(err)
		return
	}

	defer resp.Body.Close()

	if resp.StatusCode == http.StatusCreated {
		digest = resp.Header.Get(http.CanonicalHeaderKey("Docker-Content-Digest"))
		return
	}

	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		return
	}

	err = &registry_error.Error{
		StatusCode: resp.StatusCode,
		Detail:     string(b),
	}

	return
}
```