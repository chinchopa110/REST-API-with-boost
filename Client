#include <boost/beast.hpp>
#include <boost/asio/connect.hpp>
#include <boost/asio/ip/tcp.hpp>
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/json_parser.hpp>
#include <iostream>
#include <sstream>

namespace http = boost::beast::http;

class Client {
public:
    std::string get_response(const std::string& host, const std::string& target) {


        boost::asio::io_context ioc;

        boost::asio::ip::tcp::resolver resolver(ioc);
        boost::asio::ip::tcp::socket socket(ioc);

        boost::asio::connect(socket, resolver.resolve(host, "80"));

        http::request<http::string_body> req(http::verb::get, target, 11);
        req.set(http::field::host, host);
        req.set(http::field::user_agent, BOOST_BEAST_VERSION_STRING);


        http::write(socket, req);

        std::string response;
        {
            boost::beast::flat_buffer buffer;
            http::response<http::dynamic_body> res;
            http::read(socket, buffer, res);

            if (res.result() != http::status::ok) {
                throw std::runtime_error("");
            }

            response = boost::beast::buffers_to_string(res.body().data());
        }
        socket.shutdown(boost::asio::ip::tcp::socket::shutdown_both);

        return response;
    }

    std::string get_field(const std::string& json, const std::string& field) {
        std::stringstream jsonEncoded(json);
        boost::property_tree::ptree root;

        try {
            boost::property_tree::read_json(jsonEncoded, root);
        }
        catch (const boost::property_tree::json_parser_error& e) {
            std::cerr << "JSON parsing error: " << e.what() << "\n";
            return "";
        }

        try {
            return root.get<std::string>(field);
        }
        catch (const boost::property_tree::ptree_error& e) {
            std::cerr << "Field not found: " << e.what() << "\n";
            return "";
        }
    }
};

int main() {
    const std::string host = "api.openweathermap.org";

    const std::string target = "/data/2.5/weather?q=/*YOUR CITY*/&type=like&APPID=/*YOUR KEY*/";
    /*
    You can get the key after registering on:https://openweathermap.org/api
        */


    Client client;
    try {
        std::string res = client.get_response(host, target);
        std::string temp = client.get_field(res, "main.temp");
        std::cout << "Temperature(in Kelvins): " << temp << "\n" << "Temperature(in Celsius): " << stoi(temp) - 273.15 << "\n";
    } catch(const std::exception& e){
        std::cerr << "Error" << "\n";
    }
        
    return 0;
}
