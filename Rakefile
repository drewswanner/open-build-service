CONTAINER_USERID = %x(id -u)

namespace :docker do
  desc 'Build our development environment'
  task :build do
    begin
      # Setup the frontend container
      sh "docker build . -t openbuildservice/frontend --build-arg CONTAINER_USERID=#{CONTAINER_USERID}"
      # Bootstrap the app
      sh "docker-compose up -d db"
      sh "docker-compose run --no-deps --rm frontend bundle exec rake dev:bootstrap RAILS_ENV=development"
    ensure
      sh "docker-compose stop"
    end
  end

  namespace :test do
    desc 'Run our frontend tests in the docker container'
    task :frontend do
      begin
        sh "docker-compose -f docker-compose.ci.yml up --abort-on-container-exit"
      ensure
        sh "docker-compose -f docker-compose.ci.yml stop"
      end
    end

    desc 'Run our backend tests in the docker container'
    task :backend do
      begin
        sh "docker-compose run --rm -w /obs backend make -C src/backend test"
      ensure
        sh "docker-compose stop"
      end
    end

    desc 'Scan the code base for syntax/code problems'
    task :lint do
      begin
        sh "docker-compose -f docker-compose.ci.yml run --rm rspec bundle exec rake dev:bootstrap dev:lint"
      ensure
        sh "docker-compose -f docker-compose.ci.yml stop"
      end
    end
  end

  namespace :maintainer do
    desc "Rebuild all our static containers"
    task rebuild: ['rebuild:base', 'rebuild:backend', 'rebuild:frontend-base', 'rebuild:mariadb', 'rebuild:memcached'] do
    end
    namespace :rebuild do
      task :base do
        sh "docker build . -t openbuildservice/base:423 -t openbuildservice/base -f Dockerfile.423"
      end
      task :mariadb do
        sh "docker build . -t openbuildservice/mariadb:423 -t openbuildservice/mariadb -f Dockerfile.mariadb"
      end
      task :memcached do
        sh "docker build . -t openbuildservice/memcached:423 -t openbuildservice/memcached -f Dockerfile.memcached"
      end
      task 'frontend-base' do
        sh "docker build . -t openbuildservice/frontend-base:423 -t openbuildservice/frontend-base -f Dockerfile.frontend-base"
      end
      task :backend do
        sh "docker build . -t openbuildservice/backend:423 -t openbuildservice/backend -f Dockerfile.backend"
      end
    end

    desc "Rebuild and publish all our static containers"
    task publish: [:rebuild, 'publish:base', 'publish:mariadb', 'publish:memcached', 'publish:backend', 'publish:frontend-base'] do
    end
    namespace :publish do
      task :base do
        sh "docker push openbuildservice/base:423"
        sh "docker push openbuildservice/base"
      end
      task :mariadb do
        sh "docker push openbuildservice/mariadb:423"
        sh "docker push openbuildservice/mariadb"
      end
      task :memcached do
        sh "docker push openbuildservice/memcached:423"
        sh "docker push openbuildservice/memcached"
      end
      task :backend do
        sh "docker push openbuildservice/backend:423"
        sh "docker push openbuildservice/backend"
      end
      task 'frontend-base' do
        sh "docker push openbuildservice/frontend-base:423"
        sh "docker push openbuildservice/frontend-base"
      end
    end
  end
end