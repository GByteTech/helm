# GByte Tech Helm Charts

Welcome to the GByte Tech Helm Charts repository. This repository contains Helm charts for various applications and
services developed and maintained by GByte Tech.

## Adding the GByte Tech Helm Repository

To add the GByte Tech Helm repository to your Helm installation, use the following command:

```bash
helm repo add gbyte https://gbytetech.github.io/helm/
```

To update the repository and fetch the latest versions of the charts:

```bash
helm repo update
```

## Available Charts

Here's a list of the available charts in this repository:

1. [FastAPI](./charts/fastapi/README.md) - A Helm chart for deploying FastAPI applications on Kubernetes.

## Using the Charts

To install a chart from this repository, use the `helm install` command followed by the release name and the chart name.
For example:

```bash
helm install my-fastapi gbyte/fastapi
```

For more detailed information about each chart, including configuration options and usage instructions, please refer to
the README file in the respective chart's directory.

## Contributing

We welcome contributions to our Helm charts! If you'd like to contribute, please:

1. Fork this repository
2. Create a new branch for your changes
3. Make your changes and commit them
4. Push your changes to your fork
5. Create a pull request

Please ensure that your changes are well-documented and include updates to the relevant README files.

## Support

If you encounter any issues or have questions about our Helm charts, please open an issue in this repository.

## License

These Helm charts are released under the [MIT License](LICENSE).

## Credits

- [Free VPN](https://forestvpn.com)
- [VPN for teams](https://spacev.net)
- [AI Driven Real Estate](https://anysqft.com)
